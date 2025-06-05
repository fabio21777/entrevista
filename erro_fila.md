### Estratégias para lidar com erros em sistemas de filas

- **Retry com limite de tentativas:**
  Reenvia mensagens automaticamente após falhas temporárias, respeitando um número máximo de tentativas configurado para evitar sobrecarga e loops infinitos.

- **Backoff exponencial com jitter:**
  Aumenta progressivamente o tempo entre tentativas e adiciona aleatoriedade para distribuir a carga, evitando picos simultâneos que agravam falhas.

- **Dead Letter Queue (DLQ):**
  Encaminha mensagens que excederam o limite de reprocessamento para uma fila separada, permitindo auditoria, diagnóstico ou reprocessamento manual.

- **Fila de reprocessamento com atraso (delay queue):**
  Adota filas intermediárias com tempo de vida (TTL) configurado, introduzindo uma pausa controlada antes de tentar reprocessar mensagens com falhas transitórias.

- **Circuit Breaker:**
  Bloqueia temporariamente novas tentativas quando um número crítico de falhas ocorre, protegendo os sistemas downstream de colapsos e permitindo recuperação controlada.

- **Classificação de erros:**
  Distingue entre falhas transitórias (rede, timeout) e definitivas (dados inválidos), evitando reprocessar mensagens que nunca terão sucesso.

- **Isolamento de mensagens tóxicas (Poison Messages):**
  Detecta e separa mensagens que falham consistentemente por problemas estruturais ou lógicos, impedindo que comprometam a fila inteira.

- **Idempotência no processamento:**
  Garante que o reenvio de uma mesma mensagem produza sempre o mesmo efeito, evitando duplicações e efeitos colaterais indesejados.

- **Observabilidade e alertas:**
  Usa monitoramento contínuo e alarmes para detectar anomalias, acionar respostas rápidas e manter rastreabilidade sobre falhas e DLQs.
