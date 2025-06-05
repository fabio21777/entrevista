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
