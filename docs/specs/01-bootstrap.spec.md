# Spec 01 — bootstrap

## 1. Need (por quê)
- Qual dor (backlog #): #1 — Bootstrapping Spring Boot.
- A need em uma frase: "Preciso do menor monólito Spring Boot possível (Catalog + Orders
  no MySQL, um endpoint que cria um order) para ter um substrato vivo onde as próximas
  dores possam ser fabricadas."
- Meta de aprendizado / curiosidade que serve: traduzir o modelo mental de .NET/Delphi
  para o ecossistema Java/Spring, na prática.

## 2. Contexto
- Onde vive (domain/serviço): monólito único; domains Catalog e Orders (package-by-domain).
- Comportamento atual / o que existe hoje: nada — repositório só com docs.
- A dor a fabricar: nenhuma ainda (Dor #1 é fundação; ela existe para as Dores #2+ terem
  onde doer).
- Stack desta dor: Java + Spring Boot, build com Maven, MySQL nativo local (porta 3306).

## 3. Comportamento (o quê — não o como)
- Given um conjunto pequeno de Products já existentes (seed),
  When faço POST /orders com uma lista de { productId, quantity },
  Then um Order é persistido no MySQL com status inicial e o preço de cada item vem do
  Product (server-authoritative), e recebo 201 Created com a representação do Order.
- Given um productId que não existe,
  When faço POST /orders com ele,
  Then a criação é rejeitada (nada é persistido) — resposta de erro 4xx.
- Given quantity <= 0,
  When faço POST /orders,
  Then a criação é rejeitada — resposta de erro 4xx.
- Given um Order já criado,
  When faço GET /orders/{id},
  Then recebo a representação daquele Order.
- Non-goals explícitos: sem payment, sem inventory/reservation, sem auth, sem idempotency,
  sem cálculo de frete/box. Nada disso agora — cada um é uma dor futura.

## 4. Acceptance criteria (como vou saber que está pronto)
- [ ] A app Spring Boot sobe e conecta no MySQL nativo local (porta 3306).
- [ ] Existe um seed de N Products (ex.: 5) no banco ao subir.
- [ ] POST /orders cria e persiste um Order + seus OrderItems e responde:
      - status 201 Created;
      - header Location: /orders/{id};
      - body com um OrderResponse DTO (id, status, createdAt, items[{ productId, quantity,
        unitPrice }], total).
- [ ] O preço gravado em cada OrderItem vem do Product no banco, NÃO do request (o request
      só carrega productId + quantity).
- [ ] GET /orders/{id} devolve o Order criado.
- [ ] Reiniciar a app não perde os Orders criados (persistência real, não in-memory).
- Medição/experimento que prova: crio 2 orders via curl/Postman, reinicio a app, faço GET
  /orders/{id} nos dois (e confiro no MySQL) — continuam lá com o unitPrice do momento da
  criação.

## 5. Failure modes & edge cases
- MySQL fora / credencial errada no STARTUP: fail-fast — a app não sobe e loga o erro
  claro (default do HikariCP). Melhor recusar a subir do que virar um zumbi que dá 500 em
  toda chamada.
- productId inexistente no POST: rejeitar (4xx), não persistir Order parcial.
- quantity <= 0: rejeitar (4xx).
- (Consciente e fora de escopo agora):
  - Banco cai em RUNTIME (app de pé, MySQL morre no meio): degradação graciosa,
    circuit breaker, timeouts → é a Dor #11 (Resiliência) e a Dor #8 (acoplamento
    síncrono). NÃO tratamos aqui; adicionar agora roubaria o aprendizado dessas dores.
  - Submit duplicado do mesmo order → duplicata: é a Dor #10 (idempotency). NÃO tratamos.
  - Health check / observabilidade (/health): é a Dor #20. Por ora o "aviso" é o log.

## 6. Trade-offs aceitos
- Monólito + MySQL nativo + chamadas síncronas in-process: simples de propósito.
- Preço snapshot no OrderItem (imutável) em vez de reler o Product: o Order é um registro
  financeiro histórico — não pode mudar retroativamente quando o preço do Product mudar
  (mesma lógica de uma nota fiscal). Custo: um Product renomeado/reprecificado depois não
  se reflete em orders antigos (que é justamente o comportamento correto).
- Não tratar idempotency/concorrência/consistência/resiliência agora — são dores futuras;
  tratá-las aqui seria gold-plating e roubaria o aprendizado delas.
- MySQL nativo (não Docker): resetar o banco depois (Dor #2, 50–100M linhas) será mais
  chato do que matar um container — custo aceito de propósito, para não adotar Docker pela
  metade antes de ele ser uma dor deliberada.
- Devolver DTO em vez da entity JPA: um pouco mais de código (mapeamento), em troca de
  desacoplar o contrato da API do schema do banco e evitar vazamento/lazy-loading.

## 7. Design sketch (o como — curto)
- Uma app Spring Boot (Maven), package-by-domain: pacotes `catalog` e `orders`.
- Entities (JPA/Hibernate):
  - Product (id, sku, name, price[, weight, dimensions]).
  - Order (id, createdAt, status, total).
  - OrderItem (id, orderId, productId, quantity, unitPrice).  // unitPrice = snapshot
- Spring Data Repositories (~ DbContext/DbSet do EF).
- OrderController: POST /orders e GET /orders/{id} (~ [ApiController] do ASP.NET),
  retornando ResponseEntity com 201/Location (~ CreatedAtAction).
- DTOs: CreateOrderRequest { items: [{ productId, quantity }] } e OrderResponse.
- Seed de Products via script de inicialização do schema/dados.

## 8. Perguntas em aberto
- Resolvidas nesta versão (padrão REST de criação, fail-fast no startup, preço
  server-authoritative + snapshot). Novas perguntas que surgirem na implementação entram
  aqui.
```
