### Estratégias para Erros no Processamento de Arquivos

Ao processar arquivos, é fundamental adotar práticas que aumentem a resiliência e evitem falhas em cascata. As principais estratégias são:

- **[Tratamento de arquivos grandes](https://www.baeldung.com/java-read-lines-large-file):**
  Utilize leitura em streaming (`BufferedReader`, `inputStream`, `Scanner`, `BufferedReader`) e divisão por partes ([*chunking*](https://www.baeldung.com/java-read-file-split-into-several)) para evitar sobrecarga de memória.


- **Processamento total vs. parcial:**
  Escolha entre abordagem atômica (tudo ou nada) ou processamento parcial com log de falhas.
  Exemplo com Reactor:
  `.onErrorContinue((ex, obj) -> log.warn("Erro ao processar linha: {}", obj))`

- **Particionamento e processamento paralelo:**
  Divida dados em blocos lógicos e utilize paralelismo (`CompletableFuture`, `ExecutorService`, `.parallel()` no `Flux`) para ganho de desempenho.
  Controle o fluxo com operadores como `limitRate(n)` para evitar sobrecarga de threads.

- **Gerenciamento de pressão de fluxo (*backpressure*):**
  Limite a quantidade de itens processados simultaneamente (`limitRate(n)`) e use `Schedulers.boundedElastic()` para operações bloqueantes.

- **Retentativa com atraso exponencial:**
  Reaplique operações que falharam temporariamente com backoff exponencial.
  Exemplo com Reactor:
  ```java
  flux.retryBackoff(3, Duration.ofSeconds(1), Duration.ofSeconds(5))
  ```

- **Fila de mensagens com erro (DLQ customizada):**
  Redirecione dados problemáticos para análise posterior.
  Exemplo com Reactor:
  ```java
  flux.flatMap(linha -> processar(linha)
      .onErrorResume(ex -> salvarEmDLQ(linha, ex)))
  ```

- **Processamento idempotente:**
  Garanta que reprocessamentos não causem efeitos colaterais, usando chaves únicas, hash da entrada ou marcação de "já processado". Essencial para sistemas com retry, concorrência ou múltiplos workers.

**Observação:**
A biblioteca Reactor do Java facilita a implementação dessas estratégias de forma reativa, promovendo resiliência, paralelismo e controle de fluxo no processamento de arquivos.
