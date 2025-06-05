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

> **Volumetria**: Quantidade total de dados gerenciados por um sistema (armazenamento, processamento, transmissão).

Contexto: Um sistema web utilizado para gerenciar eventos e emitir certificados digitais personalizados para os participantes.

⚙️ Processamento
Geração de certificados em lote:

1.000 certificados por evento

Tempo médio para geração: 0,2s por certificado

Processamento total por evento: ~3 minutos e 20 segundos

Picos de geração simultânea:

Até 10.000 certificados/dia em períodos de alta demanda

Requer escalonamento automático de instâncias de geração


---

### Qual fila era usada?
<!-- Resposta: -->

---

### Estratégias para lidar com erros de filas
<!-- Resposta: -->

- Retentativas (envio, leitura, processamento)
- Dead Letter Queue (DLQ)
- Backoff exponencial para retentativas

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
