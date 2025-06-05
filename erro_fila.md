### Estratégias para lidar com erros em sistemas de filas

- **Retry com limite de tentativas:**
  Reenvia mensagens automaticamente após falhas temporárias, respeitando um número máximo de tentativas configurado para evitar sobrecarga e loops infinitos.
  🔗 [AWS Retry Best Practices](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)

- **Backoff exponencial com jitter:**
  Aumenta progressivamente o tempo entre tentativas e adiciona aleatoriedade para distribuir a carga, evitando picos simultâneos que agravam falhas.
  🔗 [AWS Builders’ Library – Timeouts, Retries and Backoff](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)

- **Dead Letter Queue (DLQ):**
  Encaminha mensagens que excederam o limite de reprocessamento para uma fila separada, permitindo auditoria, diagnóstico ou reprocessamento manual.
  🔗 [AWS SQS – DLQ Documentation](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)

- **Fila de reprocessamento com atraso (delay queue):**
  Adota filas intermediárias com tempo de vida (TTL) configurado, introduzindo uma pausa controlada antes de tentar reprocessar mensagens com falhas transitórias.
  🔗 [Amazon SQS Delay Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-delay-queues.html)

- **Circuit Breaker:**
  Bloqueia temporariamente novas tentativas quando um número crítico de falhas ocorre, protegendo os sistemas downstream de colapsos e permitindo recuperação controlada.
  🔗 [Microsoft Azure – Circuit Breaker Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker)


- **Idempotência no processamento:**
  Garante que o reenvio de uma mesma mensagem produza sempre o mesmo efeito, evitando duplicações e efeitos colaterais indesejados.

- **Observabilidade e alertas:**
  Usa monitoramento contínuo e alarmes para detectar anomalias, acionar respostas rápidas e manter rastreabilidade sobre falhas e DLQs.
