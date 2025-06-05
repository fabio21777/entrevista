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
