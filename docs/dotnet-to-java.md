# .NET / Delphi → Java: mapa de tradução (Pedra de Roseta)

Este doc existe para uma need específica do Hermes: você já é sênior em C#/.NET e Delphi. O
gap **não** é lógica — é o vocabulário e as convenções do ecossistema Java/Spring. Aqui a
gente traduz o que você já domina para o mundo novo, para você operar como o sênior que é,
não como quem segue tutorial.

É um doc-vivo: adicione mapeamentos conforme os encontrar (ideia da regra "escreva no
journal", consolidada aqui). Os termos técnicos ficam em inglês; a explicação em pt-BR.

## Equivalências rápidas

| .NET (C#)                 | Java / Spring                              |
|---------------------------|--------------------------------------------|
| `.csproj` / solution      | `pom.xml` (Maven) ou `build.gradle`        |
| NuGet                     | Maven ou Gradle (dependency management)    |
| ASP.NET Core Web API      | Spring Boot (Spring Web)                   |
| `[ApiController]` controller | `@RestController`                       |
| `IServiceCollection` / DI container | `ApplicationContext` + beans (`@Component`/`@Service`/`@Repository`) |
| Entity Framework / `DbContext` / `DbSet<T>` | Spring Data JPA / Hibernate (`EntityManager`, `Repository`) |
| EF Migrations             | Flyway ou Liquibase                        |
| `appsettings.json`        | `application.yml` / `application.properties` |
| LINQ                      | Streams API (`.Where()`→`.filter()`, `.Select()`→`.map()`) |
| `IHttpClientFactory` / `HttpClient` | `RestClient`/`RestTemplate` ou OpenFeign |
| Data Annotations (validação) | Bean Validation (`@NotNull`, `@Valid`)  |
| Middleware / filters      | Servlet Filters / `HandlerInterceptor` / `@RestControllerAdvice` |
| `Program.cs` / `Startup`  | `main()` com `@SpringBootApplication` + auto-configuration |
| `async`/`await`           | Virtual Threads (Project Loom, Java 21) — modelo de bloqueio barato |
| `record` (C#)             | `record` (Java 16+)                        |
| `struct` (value type)     | não há equivalente direto; use `record` (imutável) + tipos primitivos |
| xUnit / NUnit + Moq       | JUnit 5 + Mockito                          |

## Modelos mentais que valem detalhar

- **DI / IoC:** o `ServiceCollection` do ASP.NET vira o IoC container do Spring. Prefira
  **injeção via construtor** (não via campo) — é o equivalente idiomático e testável.
  Ciclo de vida de bean (`@Component`, `@Service`, `@Repository`) ~ registrar serviços no
  container.
- **ORM — estados do Hibernate:** transient / persistent / detached. Não existe assim tão
  explícito no EF; entender isso evita surpresas de "por que meu update não persistiu".
- **Lazy vs Eager fetch e o N+1 problem:** o clássico "uma query virou 1+N queries". No EF
  você já viu isso; no Hibernate ele morde igual. Reproduzir e resolver o N+1 é parte da
  Dor #5.
- **Streams vs LINQ:** mesma ideia (filter/map/reduce/collect), sintaxe diferente. `Collectors`
  ~ os métodos terminais do LINQ.
- **Virtual Threads (Project Loom):** o grande diferencial do Java 21. Onde no C# você
  pensaria `async/await` para não bloquear thread, o Java te dá threads baratíssimas que
  podem bloquear sem custo. Alavanca direta nas Dores #14 e #20 (threads empilhando,
  throughput).
- **Concorrência clássica:** `synchronized`, `lock`, `volatile`, `ExecutorService`. O
  Singleton "com synchronized" que você lembra do Java continua existindo, mas hoje raramente
  na mão.

## Ferramentas de referência (do ecossistema)

- **JDK:** LTS 21 (não perca tempo com o 8).
- **IDE:** IntelliJ IDEA.
- **Build:** Maven (padrão de mercado; escolha do Hermes) ou Gradle.
- **Banco:** MySQL (início do Hermes, de propósito), depois Postgres (Dor #12).

> Nota: mapeamentos vindos de material de estudo/roadmaps foram abstraídos aqui. Os projetos
> descartáveis desses roadmaps (CSV processor, cashback, etc.) **não** são usados — o Hermes
> pratica nas próprias fatias orientadas a dor, nunca em CRUDzinho de tutorial.
