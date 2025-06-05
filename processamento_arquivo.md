### Estratégias para Erros no Processamento de Arquivos

Ao processar arquivos, é fundamental adotar práticas que aumentem a resiliência e evitem falhas em cascata. As principais estratégias são:

- **Tratamento de arquivos grandes:**
  Utilize leitura em streaming (`BufferedReader`, `Files.lines`, `StAX`, `DataBufferUtils`) e divisão por partes (*chunking*) para evitar sobrecarga de memória.

- **Detecção e tratamento de arquivos corrompidos:**
  Implemente verificação de integridade (checksum, ex.: SHA-256), validação de assinatura (headers, *magic bytes*) e degradação graciosa, processando apenas as partes válidas e registrando erros.

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
