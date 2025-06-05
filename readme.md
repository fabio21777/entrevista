# Detalhes Técnicos

## Filas

### Tipos de Filas

#### Fila Padrão (Standard)

![alt text](image.png)

- **Alta taxa de transferência:** Suporta grande volume de mensagens por segundo.
- **Entrega pelo menos uma vez:** Pode haver duplicidade, exige idempotência no processamento.
- **Ordenação de melhor esforço:** Não garante ordem de chegada.
- **Alta durabilidade:** Mensagens replicadas em múltiplas zonas.
- **Tempo de visibilidade configurável:** Controla quando a mensagem pode ser reprocessada.

**Quando usar?**

- Processamento assíncrono de grandes volumes.
- Distribuição de tarefas entre múltiplos workers.
- Processamento em lote sem necessidade de ordem.

#### Fila FIFO

![alt text](image-1.png)

- **Ordem garantida:** Mensagens processadas na ordem de envio dentro de grupos.
- **Processamento único:** Sem duplicidade, com deduplicação automática.
- **Rendimento controlado:** Até 3.000 mensagens/s em lote, 300/s individual.

**Quando usar?**
- Ordem dos eventos é essencial (ex: comandos sequenciais, atualização de preços, matrícula em etapas).

---

## Volumetria

> **Volumetria:** Quantidade total de dados gerenciados por um sistema (armazenamento, processamento, transmissão).

**Contexto:**
Sistema web para gerenciar eventos e emitir certificados digitais personalizados.

- Geração em lote: cerca de 1.000 certificados por evento
- Tempo médio de geração: 0,2s por certificado (~3min20s por evento)
- Pico de demanda: até 10.000 certificados/dia em períodos de alta
- Necessidade de escalonamento automático das instâncias de geração

---

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

---

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

---

## Mensageria

### O que é mensageria?
<!-- Resposta: -->

---

### Principais conceitos de mensageria
<!-- Resposta: -->

---

### Ferramentas de mensageria conhecidas
<!-- Resposta: -->

---

## HTTP - Principais conceitos

### Principais verbos HTTP
<!-- Resposta: -->

---

### Principais códigos de status HTTP
<!-- Resposta: -->

---

### O que é idempotência e relação com HTTP?
<!-- Resposta: -->

---

## Banco de Dados

### O que é um banco de dados relacional? Quando usar?
<!-- Resposta: -->

---

#### ACID - O que é e como funciona?
<!-- Resposta: -->

---

### O que é um banco de dados não relacional? Quando usar?
<!-- Resposta: -->

---

#### Relação entre bancos relacionais e não relacionais
<!-- Resposta: -->

---

## Novidades

### Novas features do Java e como usar
<!-- Resposta: -->

---

### Novas features do Postgres e como usar
<!-- Resposta: -->

---

### Diferenças entre Spring Boot e frameworks nativos (Quarkus, Micronaut)
<!-- Resposta: -->
