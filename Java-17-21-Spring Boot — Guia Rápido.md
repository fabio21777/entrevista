# Java 17 / 21 + Spring Boot — Guia Rápido

Foco em LTS: **Java 17**, **Java 21** e versões marco do **Spring Boot**.

---

## Java 17 LTS

### Records
DTOs imutáveis sem boilerplate. Geram construtor, getters, `equals`, `hashCode`, `toString`.

```java
public record Cliente(String nome, String email) {}

var c = new Cliente("Fabio", "fabio@email.com");
c.nome();   // "Fabio"
c.email();  // "fabio@email.com"
```

### Sealed Classes
Hierarquias fechadas. Só os tipos listados em `permits` podem estender.

```java
public sealed interface Pagamento
    permits Pix, Boleto, Cartao {}

public record Pix(String chave, BigDecimal valor)        implements Pagamento {}
public record Boleto(String codigo, BigDecimal valor)    implements Pagamento {}
public record Cartao(String numero, BigDecimal valor)    implements Pagamento {}
```

Filhos podem ser `final`, `sealed` (continua fechado) ou `non-sealed` (abre de novo).

### Pattern Matching para `instanceof`
Fim do cast manual.

```java
// Antes
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// Java 17
if (obj instanceof String s) {
    System.out.println(s.length());
}
```

### Switch Expressions
Switch como expressão, com `->`, sem fall-through.

```java
String dia = switch (numero) {
    case 1, 7 -> "Fim de semana";
    case 2, 3, 4, 5, 6 -> "Dia útil";
    default -> "Inválido";
};
```

### Text Blocks
Strings multilinha sem concatenação.

```java
String sql = """
    SELECT id, nome
    FROM cliente
    WHERE ativo = true
    """;
```

### NPE "helpful"
A mensagem aponta exatamente qual variável estava null:
```
Cannot invoke "Cliente.email()" because "pedido.cliente" is null
```

---

## Java 21 LTS

### Virtual Threads
Threads leves gerenciadas pela JVM. Milhões em uma JVM. Ideal para I/O-bound.

```java
// Pool tradicional: caro, limitado
try (var executor = Executors.newFixedThreadPool(200)) { ... }

// Java 21: virtual threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10_000).forEach(i ->
        executor.submit(() -> chamarApiExterna(i))
    );
}
```

### Pattern Matching para `switch`
Switch sobre tipos, com guards (`when`) e exhaustiveness.

```java
public BigDecimal calcularTaxa(Pagamento p) {
    return switch (p) {
        case Pix pix                       -> BigDecimal.ZERO;
        case Boleto b when b.vencido()     -> new BigDecimal("5.00");
        case Boleto b                      -> new BigDecimal("2.50");
        case Cartao c                      -> c.valor().multiply(new BigDecimal("0.03"));
        // sem default! sealed garante exhaustiveness
    };
}
```

### Record Patterns
Desestruturação de records dentro de pattern matching.

```java
if (obj instanceof Cartao(var numero, var valor)) {
    System.out.println("Cartão " + numero + " no valor de " + valor);
}

// Em switch:
return switch (pagamento) {
    case Pix(var chave, var valor)         -> "Pix de " + valor + " para " + chave;
    case Cartao(var n, var v) when v.compareTo(BigDecimal.valueOf(1000)) > 0
                                            -> "Cartão acima de 1k";
    case Cartao(var n, var v)              -> "Cartão comum";
    case Boleto b                          -> "Boleto";
};
```

### Sequenced Collections
`getFirst()`, `getLast()`, `reversed()` em coleções ordenadas.

```java
List<String> lista = List.of("a", "b", "c");
lista.getFirst();   // "a"
lista.getLast();    // "c"
lista.reversed();   // [c, b, a]
```

### Generational ZGC
ZGC agora geracional, mais eficiente com objetos de vida curta.

```bash
java -XX:+UseZGC -XX:+ZGenerational -jar app.jar
```

---

## Spring Boot — versões marco

### Spring Boot 2.7 (mai/2022)
Última da linha 2.x. Ainda com `javax.*`. Ponto de partida da maioria das migrações.

### Spring Boot 3.0 (nov/2022) — marco geracional

**Java 17 baseline obrigatório.**
**`javax.*` → `jakarta.*`** — maior atrito real em upgrade.

**`@HttpExchange`** — clients declarativos nativos:
```java
public interface ClienteApi {
    @GetExchange("/clientes/{id}")
    Cliente buscar(@PathVariable Long id);
}
```

**Problem Details (RFC 7807)** — erros REST padronizados:
```java
@ExceptionHandler(ClienteNaoEncontradoException.class)
public ProblemDetail handle(ClienteNaoEncontradoException ex) {
    return ProblemDetail.forStatusAndDetail(
        HttpStatus.NOT_FOUND, ex.getMessage()
    );
}
```

**Micrometer Observation API** — substitui Sleuth, unifica metrics + tracing.

**GraalVM Native Image** como cidadão de primeira classe (startup em ms).

### Spring Boot 3.2 (nov/2023) — casa com Java 21

**Virtual Threads via flag:**
```yaml
spring:
  threads:
    virtual:
      enabled: true
```
Tomcat, `@Async`, scheduler passam a usar virtual threads automaticamente.

**`RestClient`** — substituto moderno do `RestTemplate`:
```java
RestClient client = RestClient.create();

Cliente c = client.get()
    .uri("https://api.exemplo.com/clientes/{id}", 1)
    .retrieve()
    .body(Cliente.class);
```

**`JdbcClient`** — alternativa fluente ao `JdbcTemplate`:
```java
List<Cliente> clientes = jdbcClient
    .sql("SELECT * FROM cliente WHERE ativo = :ativo")
    .param("ativo", true)
    .query(Cliente.class)
    .list();
```

**CRaC** — startup quase instantâneo via snapshot da JVM.

### Spring Boot 3.4 (nov/2024)
- **Structured logging nativo** (JSON out of the box)
- Melhorias em CDS (Class Data Sharing) reduzindo startup
- `@Fallback` para beans

---

## Exemplo integrado

Um único exemplo conectando os principais recursos do Java 21 + Spring Boot 3.2.

```java
// === DOMÍNIO: sealed + records ===
public sealed interface Pagamento
    permits Pagamento.Pix, Pagamento.Boleto, Pagamento.Cartao {

    BigDecimal valor();

    record Pix(String chave, BigDecimal valor) implements Pagamento {}
    record Boleto(String codigo, BigDecimal valor, LocalDate vencimento) implements Pagamento {}
    record Cartao(String numero, BigDecimal valor, int parcelas) implements Pagamento {}
}

// === SERVIÇO: pattern matching + record patterns + guards ===
@Service
public class TaxaService {

    public BigDecimal calcular(Pagamento p) {
        return switch (p) {
            case Pagamento.Pix pix                       -> BigDecimal.ZERO;
            case Pagamento.Boleto b when b.vencimento()
                    .isBefore(LocalDate.now())           -> new BigDecimal("5.00");
            case Pagamento.Boleto b                      -> new BigDecimal("2.50");
            case Pagamento.Cartao(var n, var v, var p)
                    when p > 1                           -> v.multiply(new BigDecimal("0.05"));
            case Pagamento.Cartao c                      -> c.valor().multiply(new BigDecimal("0.03"));
        };
    }
}

// === CLIENT EXTERNO: @HttpExchange ===
public interface AntifraudeApi {
    @PostExchange("/analisar")
    ResultadoAnalise analisar(@RequestBody Pagamento p);
}

// === CONTROLLER: Problem Details ===
@RestController
@RequestMapping("/pagamentos")
public class PagamentoController {

    private final TaxaService taxaService;
    private final AntifraudeApi antifraude;

    public PagamentoController(TaxaService t, AntifraudeApi a) {
        this.taxaService = t;
        this.antifraude = a;
    }

    @PostMapping
    public Recibo processar(@RequestBody Pagamento p) {
        var analise = antifraude.analisar(p);
        if (!analise.aprovado()) {
            throw new PagamentoRejeitadoException("Reprovado pelo antifraude");
        }
        return new Recibo(p, taxaService.calcular(p));
    }

    @ExceptionHandler(PagamentoRejeitadoException.class)
    public ProblemDetail handle(PagamentoRejeitadoException ex) {
        return ProblemDetail.forStatusAndDetail(
            HttpStatus.UNPROCESSABLE_ENTITY,
            ex.getMessage()
        );
    }
}

// === application.yml ===
// spring:
//   threads:
//     virtual:
//       enabled: true   <- todas as requisições rodam em virtual threads
```

---

## Resposta de 60 segundos

> Em Java, foco em dois LTS: o **17** consolidou Records, Sealed Classes, Pattern Matching e Switch Expressions — código mais expressivo e seguro. O **21** trouxe Virtual Threads, que muda o jogo de concorrência em microsserviços I/O-bound, e Pattern Matching para switch combinado com Record Patterns, fechando o ciclo de programação algébrica em Java.
>
> No Spring, o marco é o **Boot 3.0**: baseline Java 17, migração `javax` → `jakarta`, Native Image como cidadão de primeira classe e Observation API unificando observabilidade. O **3.2** é onde a integração com Java 21 fica plena, com suporte a Virtual Threads via flag e o novo `RestClient` substituindo o `RestTemplate`.
