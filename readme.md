# Detalhes Técnicos

## Filas

### [Tipos de Filas aws](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-queue-types.html)

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

## [Volumetria](https://witify.io/en/blog/volumetric-software-planning-and-managing-your-data/)

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

Mensageria é um padrão de comunicação entre sistemas que permite a troca assíncrona de mensagens, desacoplando produtores e consumidores. Facilita a escalabilidade, resiliência e flexibilidade na integração de aplicações. É um dos pilares de sistemas autônomos e reativos, sendo caracterizada por:

1. Responsividade
2. Elasticidade
3. Resiliência
4. Orientação a mensagens/eventos

---

### Principais conceitos de mensageria

![alt text](image-2.png)

Os sistemas de mensageria funcionam basicamente com três componentes principais:

- **Produtor:** Componente que envia mensagens para a fila ou tópico.
- **Consumidor:** Componente que recebe e processa mensagens da fila ou tópico.
- **Tópico/Fila:** Canal de comunicação onde mensagens são publicadas e consumidas. Permite múltiplos consumidores.

Além disso, há padrões como *publish/subscribe* (publicação/assinatura), onde múltiplos consumidores podem receber a mesma mensagem, e *point-to-point* (ponto a ponto), onde cada mensagem é consumida por apenas um consumidor.

Exemplo:
No Kafka, temos produtores publicando mensagens em tópicos, consumidores lendo desses tópicos e o broker intermediando a comunicação.

---

---

---

### [Principais Códigos de Status HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Reference/Status)

Os códigos de status HTTP indicam o resultado de uma requisição feita a um servidor. Eles são organizados por classes numéricas:

#### 🔵 1xx - Informativo

* **100 Continue**: O cliente pode continuar com a requisição.
* **101 Switching Protocols**: O servidor está mudando de protocolo, conforme solicitado.

#### 🟢 2xx - Sucesso

* **200 OK**: Requisição bem-sucedida.
* **201 Created**: Um novo recurso foi criado.
* **202 Accepted**: A requisição foi aceita, mas ainda não processada.
* **204 No Content**: A requisição foi bem-sucedida, sem conteúdo de resposta.

#### 🟡 3xx - Redirecionamento

* **300 Multiple Choices**: Existem múltiplas opções para o recurso solicitado.
* **301 Moved Permanently**: O recurso foi movido de forma permanente.
* **302 Found**: O recurso foi encontrado em outra URL temporariamente.
* **303 See Other**: Redirecionamento para outro recurso.
* **304 Not Modified**: O recurso não foi modificado desde a última solicitação.
* **307 Temporary Redirect**: Redirecionamento temporário mantendo o método original.
* **308 Permanent Redirect**: Redirecionamento permanente mantendo o método original.

#### 🔴 4xx - Erro do Cliente

* **400 Bad Request**: Erro na requisição do cliente.
* **401 Unauthorized**: Autenticação necessária.
* **403 Forbidden**: Acesso ao recurso negado.
* **404 Not Found**: Recurso não encontrado.
* **405 Method Not Allowed**: Método HTTP não permitido para esse recurso.
* **409 Conflict**: Conflito no estado atual do recurso.
* **422 Unprocessable Entity**: Requisição bem formada, mas não processável.
* **429 Too Many Requests**: Limite de requisições excedido.

#### ⚫ 5xx - Erro do Servidor

* **500 Internal Server Error**: Erro interno no servidor.
* **501 Not Implemented**: Método não implementado pelo servidor.
* **502 Bad Gateway**: Resposta inválida de um gateway.
* **503 Service Unavailable**: Serviço temporariamente indisponível.
* **504 Gateway Timeout**: Timeout ao esperar resposta de outro servidor.

---

### [Verbos HTTP e Idempotência](https://developer.mozilla.org/pt-BR/docs/Glossary/Idempotent)

A **idempotência** é uma propriedade que garante que múltiplas execuções da mesma requisição produzem o mesmo efeito no servidor.

#### [Verbos HTTP comuns](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Reference/Methods):

* **GET**: Recupera dados de um recurso. ✅ Idempotente.
* **POST**: Envia dados para criação de um novo recurso. ❌ Não idempotente(porém é possivel torná-lo idempotente com práticas específicas como uuid no cliente side).
* **PUT**: Atualiza ou cria um recurso com um identificador específico. ✅ Idempotente.
* **PATCH**: Atualiza parcialmente um recurso. ⚠️ Nem sempre idempotente(pode existe contadores e modificares de estado para cada atualização).
* **DELETE**: Remove um recurso. ✅ Idempotente (assumindo deleção repetida não causa erro).
* **HEAD**: Retorna apenas os headers do recurso. ✅ Idempotente(Imagine que você quer saber se uma imagem existe e qual o tamanho dela, mas sem baixá-la.).
* **OPTIONS**: Retorna os métodos permitidos. ✅ Idempotente(O método OPTIONS é usado para descobrir quais métodos HTTP são suportados por um recurso ou servidor. Ele é muito utilizado em CORS (Cross-Origin Resource Sharing)).
* **CONNECT**: Estabelece um túnel TCP (usado com HTTPS- CONNECT é usado principalmente para estabelecer um túnel TCP entre o cliente e o servidor, permitindo a comunicação segura, como em conexões HTTPS através de proxies.). ❌ Não idempotente.
* **PURGE**: (usado por alguns proxies) Limpa cache. ❌ Não idempotente.

#### Por que importa?

* **Requisições idempotentes** são seguras para reexecução em falhas de rede.
* **Servidores e proxies** podem aplicar caching, retries e roteamento confiável com base nessa característica.

---

## Banco de Dados

### O que é um banco de dados relacional? Quando usar?


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
