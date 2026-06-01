# Virtual Threads (Java 21) — Revisão de Entrevista

> Tema "diferencial" pra dev pleno. Saber explicar **o mecanismo**, **quando usar**, **quando NÃO** e as **pegadinhas**.

## Pitch de uma frase

Virtual threads dão a **escalabilidade do modelo reativo** com o **modelo de programação simples e bloqueante** de sempre.

---

## O problema que resolvem

No modelo clássico (thread-per-request), cada requisição pega uma **platform thread** = wrapper 1:1 de uma thread do SO. Thread do SO é cara: ~1MB de stack + custo de troca de contexto no kernel. Um servidor aguenta só uns poucos milhares.

E o desperdício: numa API típica, a thread passa a **maior parte do tempo bloqueada esperando I/O** (banco, outro serviço, fila). Recurso caro parado. Foi isso que empurrou todo mundo pro reativo (WebFlux, `CompletableFuture`) — que escala, mas custa caro em legibilidade e debug (callbacks, stack trace inútil).

Virtual threads resolvem os dois lados.

---

## Mecanismo (o pulo do gato)

Virtual thread é uma thread leve gerenciada pela **JVM**, não pelo SO. Não tem thread do SO dedicada. Existe um pool pequeno de **carrier threads** (platform threads reais, ~uma por core). Muitas virtual threads rodam por cima de poucas carriers — modelo **M:N**.

Ciclo **mount / unmount**:

1. Virtual thread com trabalho → é **montada** numa carrier e executa.
2. Bloqueou em I/O → a JVM **desmonta** ela da carrier e guarda o estado no **heap**. A carrier fica livre na hora pra montar **outra** virtual thread.
3. I/O terminou → virtual thread é **remontada** (em qualquer carrier livre) e continua de onde parou.

Resultado: `Thread.sleep()` ou bloqueio numa query **não trava a thread do SO** — só "estaciona" a virtual thread baratinho no heap. Por isso você escreve código bloqueante normal e mesmo assim escala.

```
[ heap: VTs estacionadas, esperando I/O, custo ~0 ]
        |  monta (tem trabalho)     ^  desmonta (bloqueou em I/O)
        v                           |
[ poucas carrier threads (≈ nº de cores) ] ← cada uma roda 1 VT por vez
```

---

## Quando USAR

Regra: **I/O-bound + muita concorrência.** Tarefas que passam o tempo *esperando*, não calculando.

- APIs/microsserviços que fazem muita query, chamam outros serviços, leem fila.
- Quando quer **escala sem migrar pra reactive** — mantém `RestClient`/JDBC bloqueante, código linear e legível, e aguenta dezenas de milhares de requisições concorrentes.
- Crawlers, agregadores, fan-out de chamadas paralelas.

Jeito idiomático: **uma virtual thread por tarefa**, com `Executors.newVirtualThreadPerTaskExecutor()`.
No Spring Boot 3.2+: `spring.threads.virtual.enabled=true` e o Tomcat atende cada request numa virtual thread.

---

## Quando NÃO usar (a parte que cai em entrevista)

- **CPU-bound.** Cálculo pesado (imagem, compressão, cripto em loop) não ganha nada — você continua limitado pelos cores. Use pool de platform threads do tamanho dos cores.
- **Não pode poolar.** VTs são feitas pra criar/descartar em massa. Pool fixo (`newFixedThreadPool`) **destrói o benefício** e limita a concorrência. Pool de VT = anti-padrão.
- **Baixa concorrência.** Poucas requisições simultâneas → platform threads já resolviam, sem ganho perceptível.
- **Quando você usava o pool como controle de carga.** O pool antigo servia, sem querer, de *rate limiter* (só N por vez). Com VTs "ilimitadas" você pode estourar um recurso externo (pool de conexões do banco, API de terceiro). Use **`Semaphore`** ou limitador explícito — não conte com o pool.

---

## Pegadinhas técnicas (pra impressionar)

**Pinning.** Se a VT entra num bloco `synchronized` e bloqueia ali dentro, ela fica **presa (pinned)** na carrier e não desmonta — perde o benefício. Recomendação histórica: trocar `synchronized` por `ReentrantLock` nos pontos críticos. Citar que isso foi **bastante mitigado** nas versões recentes do JDK (pinning por `synchronized` deixou de ser problema), mas o conceito ainda cai.

**ThreadLocal.** Com milhões de VTs, `ThreadLocal` pesa e pode vazar memória. É o motivo dos **Scoped Values** (final no Java 25): substituto imutável, com escopo (try-with), que propaga corretamente. Boa amarrar os dois temas.

---

## Comparativo de código

**Antes — pool de platform threads (limitado):**
```java
// Só ~200 tarefas concorrentes de verdade; o resto fica na fila
ExecutorService exec = Executors.newFixedThreadPool(200);
for (var pedido : pedidos) {
    exec.submit(() -> {
        var dados = clienteHttp.buscar(pedido); // bloqueia thread do SO (cara)
        repo.salvar(dados);                      // bloqueia de novo
    });
}
```

**Depois — virtual thread por tarefa (escala):**
```java
// Milhares de tarefas concorrentes; bloqueio em I/O só "estaciona" a VT
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {
    for (var pedido : pedidos) {
        exec.submit(() -> {
            var dados = clienteHttp.buscar(pedido); // bloqueia? desmonta a VT, libera a carrier
            repo.salvar(dados);
        });
    }
} // try-with-resources espera todas terminarem
```

Mesmo código bloqueante e legível — só muda o executor.

---

## Resposta de 30 segundos (decore o sentido, não a letra)

> "Virtual threads são threads leves gerenciadas pela JVM, não pelo SO. Muitas rodam sobre poucas carrier threads. Quando bloqueiam em I/O, a JVM desmonta a virtual e libera a carrier pra outra, então eu escrevo código bloqueante simples e mesmo assim escalo pra milhares de tarefas concorrentes. Servem pra I/O-bound, não pra CPU-bound, e nunca devem ser pooladas — uma por tarefa."

Se ainda emendar **pinning** e **Scoped Values** quando provocado, você está acima da média de pleno.

---

## Checklist de revisão rápida

- [ ] Sei explicar mount/unmount e o papel das carrier threads.
- [ ] Sei dizer por que I/O-bound ganha e CPU-bound não.
- [ ] Sei que não se poola VT (e por quê).
- [ ] Sei que perdi o rate-limiting "de graça" do pool → uso `Semaphore`.
- [ ] Sei o que é pinning e como evitava (`ReentrantLock`).
- [ ] Conecto com Scoped Values no lugar de ThreadLocal.
- [ ] Sei ligar no Spring Boot (`spring.threads.virtual.enabled=true`).
