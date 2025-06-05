# Detalhes Técnicos

## Filas

### Tipos de Filas

#### Fila Padrão (Standard)

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

### Estratégias para lidar com erros de filas

- **Retry com controle de tentativas:**
  Reenvio automático de mensagens que falharam, com limite máximo de tentativas configurável.

- **Backoff exponencial (com ou sem jitter):**
  Aumenta o tempo de espera entre tentativas de forma crescente (e, opcionalmente, aleatória), evitando sobrecarga em serviços instáveis.

- **Dead Letter Queue (DLQ):**
  Armazena mensagens que excederam o número de tentativas, permitindo análise e reprocessamento posterior.

- **Retry com delay via fila temporária:**
  Utiliza uma fila intermediária com TTL (Time-To-Live) para reprocessar mensagens após um intervalo de tempo.

- **Circuit Breaker:**
  Interrompe temporariamente o processamento quando há um número elevado de falhas, protegendo o sistema de colapsos em cascata.

- **Classificação de erros:**
  Diferencia erros transitórios (passíveis de retry) de erros definitivos (como dados inválidos), evitando tentativas desnecessárias.

- **Tratamento de mensagens tóxicas (Poison Messages):**
  Isola mensagens que causam falhas sistemáticas, impedindo que travem o sistema de processamento.

- **Idempotência:**
  Se necessário, garante que o processamento de uma mensagem não cause efeitos colaterais indesejados, mesmo se for reprocessada.

- **Monitoramento e alertas:**
-
  Observabilidade sobre falhas e DLQs para resposta proativa e visibilidade de anomalias.
---

### Estratégias para erros no processamento de arquivos
<!-- Resposta: -->

- Como tratar arquivos grandes?
- O que fazer com arquivos corrompidos?
- Processamento: tudo ou nada ou parcial?
- Particionamento, paralelismo e controle de fluxo

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
