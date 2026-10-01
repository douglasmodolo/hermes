# Pain Backlog — o coração do Hermes

Esta é a lista ordenada de **dores a fabricar e bater**. Cada dor é uma *need* pronta: um
problema concreto que você provoca de propósito, sente, e então resolve. É o oposto de
estudar no vácuo.

**Como usar este arquivo**
1. Escolha uma dor (grosso modo de cima para baixo, mas siga as suas needs reais).
2. Escreva a spec dela primeiro — veja `spec-driven-development.md`.
3. **Fabrique** a dor: leve o sistema a um estado em que ela realmente dói.
4. **Observe/meça** — veja a falha com os seus próprios olhos (latência, lock, erro,
   queue).
5. **Resolva.** Depois registre no `progress-journal.md`.
6. Note a dica "→ puxa a próxima": uma dor naturalmente semeia a seguinte.

Cada dor lista: o tópico que você aprende, a dor a fabricar, onde ela vive, e uma ideia de
implementação (uma direção de partida — **não** uma solução para copiar).

---

## Lente de carreira (por que esta ordem)

Além da curiosidade, o Hermes serve a uma meta concreta: **vagas de backend Java em
Campinas** (perfil típico: Java 21 + Spring Boot, microserviços, mensageria, AWS,
Postgres, testes, CI/CD). Conseguir a vaga também é uma *need* real — então a ordem foi
ajustada (2026-09-30) para o repositório provar o núcleo dessas vagas **cedo**, sem
abandonar a regra de que nada entra sem uma dor:

- **Fase 0** ganhou testes, Docker e CI (o núcleo que um avaliador confere em 30 segundos)
  e segurança de API.
- **Fase 1** ganhou arquitetura hexagonal e trouxe a **migração para Postgres** para o fim
  da fase (antes ficava na Fase 4).
- **Fase 2** ganhou o **deploy na AWS** logo depois da mensageria.
- **Fase 4** ganhou MongoDB, com um uso que se justifica (não Mongo por Mongo).

| Requisito das vagas | Dor(es) |
|---|---|
| Java 21, Spring Boot, REST | 1 |
| JUnit, Mockito, TDD, BDD-lite (Given/When/Then) | 2 |
| Docker, docker-compose, Testcontainers | 3 |
| CI/CD (GitHub Actions; conceitos transferem para Jenkins/GitLab CI) | 4, 17 |
| Segurança de APIs (auth, JWT, OWASP) | 7 |
| DDD, hexagonal / Clean Architecture | 10, 11 |
| PostgreSQL | 12 |
| Microserviços | 13, 14 |
| Kafka / RabbitMQ / SQS | 15, 16, 17 |
| AWS (ECS/Fargate, RDS, S3, SQS/SNS, CloudWatch) | 17 |
| Redis | 6 |
| MongoDB | 26 |
| Observabilidade, Canary/Blue-Green, feature toggle | 28, 30 |
| Terraform / IaC, Kubernetes | 30, 32 |

---

## Fase 0 — Fundações (colocar o monólito para respirar, com rede de segurança)

### Dor 1 — Bootstrapping Spring Boot
- **Tópico:** estrutura de projeto Java + Spring Boot, a partir de uma mentalidade
  legado/.NET.
- **Fabricar:** nada ainda — suba um monólito com Catalog + Orders no MySQL, um endpoint
  que cria um order.
- **Where:** Catalog, Orders.
- **Ideia de implementação:** uma única app Spring Boot, package-by-domain (não
  por camada). Mapeie os conceitos .NET que você conhece (DI container, controllers, EF)
  para Spring (ApplicationContext, @RestController, JPA/Hibernate) — registre os
  mapeamentos em `dotnet-to-java.md`.
- **→ puxa a próxima:** agora você tem código; tente mudá-lo sem rede.

### Dor 2 — Refactor sem rede (testes automatizados)
- **Tópico:** testes automatizados — JUnit 5, Mockito, AssertJ, test slices
  (`@WebMvcTest`, `@DataJpaTest`), TDD.
- **Fabricar:** mude uma regra de Orders (ex.: o cálculo do total ou a validação de
  quantity) e quebre, sem perceber, outro comportamento da spec 01.
- **Observe:** quanto tempo e quantos curls você leva para descobrir a regressão na mão — e
  quantas você nem descobre.
- **Where:** Catalog, Orders.
- **Ideia de implementação:** transforme cada Given/When/Then da spec 01 em um teste (é o
  BDD-lite que as specs já te dão). Unit tests para as regras, com Mockito só nos
  colaboradores; slice tests para o controller (201, `Location`, 4xx de validação) e para o
  repository. Escreva o próximo comportamento em TDD (red → green → refactor).
- **Aprofundar (mock demais):** um teste que só verifica chamadas de mock quebra a cada
  refactor e não pega bug. Aprenda quando usar mock e quando usar o objeto real.
- **→ puxa a próxima:** os testes de integração precisam de um MySQL de verdade, e ele só
  existe na sua máquina.

### Dor 3 — "Na minha máquina funciona" (Docker + Testcontainers)
- **Tópico:** containers, Docker, docker-compose, Testcontainers.
- **Fabricar:** rode a suíte num ambiente sem MySQL (ou com outra versão/credencial) e veja
  falhar. Antecipe a Dor 5: imagine resetar 50–100M linhas num MySQL nativo, várias vezes.
- **Observe:** testes que dependem do estado da sua máquina; quanto tempo leva para recriar
  o banco do zero.
- **Where:** cross-cutting.
- **Ideia de implementação:** Dockerfile da app (multi-stage), docker-compose com o MySQL
  (que depois recebe Redis, Kafka...), Testcontainers nos testes de integração para ter um
  MySQL real e descartável. A spec 01 adiou o Docker de propósito — é aqui que ele deixa de
  ser "porque é importante" e vira dor.
- **Aprofundar (H2 vs Testcontainers):** H2 é mais rápido, mas mente — é outro dialeto
  SQL. Sinta um teste passar no H2 e falhar no MySQL.
- **→ puxa a próxima:** os testes existem, mas nada obriga ninguém a rodá-los.

### Dor 4 — Main quebrada (CI)
- **Tópico:** CI/CD, GitHub Actions, quality gates, branch protection.
- **Fabricar:** abra um PR que quebra um teste (ou nem compila) e faça merge na main.
- **Observe:** a main quebrada, descoberta só no próximo clone ou na próxima feature.
- **Where:** cross-cutting (entrega).
- **Ideia de implementação:** workflow do GitHub Actions com build + testes (Testcontainers
  roda no runner) em todo PR; branch protection exigindo checks verdes; cache do Maven;
  Maven Wrapper commitado. O CD (deploy automático) fica para a Dor 17.
- **Aprofundar (Jenkins/GitLab CI):** as vagas citam Jenkins e GitLab CI. Os conceitos
  (stages, artifacts, gates) se transferem — registre o mapeamento como fez com .NET→Java.
- **→ puxa a próxima:** agora você muda com confiança; faça os dados doerem.

### Dor 5 — Queries lentas em escala (indexing)
- **Tópico:** performance de banco, indexes, query plans.
- **Fabricar:** insira 50–100M de products/orders falsos. Rode a search mais comum.
- **Observe:** veja o P99 subir de dezenas de ms para segundos; leia a saída do `EXPLAIN`.
- **Where:** Catalog.
- **Ideia de implementação:** escreva um script gerador de dados; meça uma query
  antes/depois de adicionar um index; aprenda a ler o plano de execução. Resista a adicionar
  um index às cegas — primeiro *veja* o full scan. Com o banco num container (Dor 3),
  resetar é descartar um volume.
- **Aprofundar (N+1 problem):** reproduza também o clássico do ORM — uma query que vira
  1+N queries por causa de lazy fetch. Você já viu isso no EF; o Hibernate morde igual.
  Veja o N+1 no log de SQL antes de resolvê-lo (fetch join, `@EntityGraph`, batch).
- **→ puxa a próxima:** indexes ajudam mas o DB ainda está quente → cache.

### Dor 6 — Caching & invalidation
- **Tópico:** caching, TTLs, invalidation, cache stampede.
- **Fabricar:** martele a query quente centenas de vezes/segundo; veja o CPU do DB
  disparar.
- **Observe:** compare a latência de hit vs miss; então mude o preço de um product e pegue
  stale reads.
- **Where:** Catalog.
- **Ideia de implementação:** introduza Redis como um read cache com um TTL. A lição real é
  a invalidation: o que acontece quando o dado subjacente muda? Crie de propósito um bug de
  stale-read e então conserte.
- **Aprofundar (TTL por volatilidade):** o TTL não é único. Dado que quase não muda
  (metadados de product — nome, descrição) aceita TTL longo; dado volátil (price, stock)
  exige TTL curto ou invalidation ativa. Modele os dois e sinta a diferença.
- **→ puxa a próxima:** o Catalog está rápido, mas a API está escancarada.

### Dor 7 — Qualquer um cria (e lê) qualquer order (segurança de API)
- **Tópico:** Spring Security, autenticação vs autorização, JWT / OAuth2 resource server,
  OWASP Top 10.
- **Fabricar:** hoje qualquer um cria order, e `GET /orders/{id}` com id sequencial deixa
  ler o order de qualquer pessoa (IDOR).
- **Observe:** enumere ids num loop e leia orders alheios.
- **Where:** Identity, Orders.
- **Ideia de implementação:** domain Identity (signup/login) emitindo JWT; a app como
  resource server com Spring Security; o order passa a ter dono; o GET só devolve o order
  ao dono.
- **Aprofundar (OWASP):** broken access control/IDOR (o #1 do OWASP), mass assignment (por
  que o DTO da spec 01 já te protegeu), brute force no login. Trade-off: JWT stateless vs
  session (como revogar um token?).
- **→ puxa a próxima:** orders têm dono; e quando dois donos querem a última unidade?

---

## Fase 1 — Integridade de order, concorrência & design

### Dor 8 — Overselling sob concorrência
- **Tópico:** concorrência, race conditions, estratégias de locking.
- **Fabricar:** dispare N compras simultâneas da última unidade em stock.
- **Observe:** o stock fica negativo — você vendeu mais do que tinha.
- **Where:** Inventory.
- **Ideia de implementação:** reproduza a race primeiro (read-then-write ingênuo). Então
  explore pessimistic vs optimistic locking e compare. Entenda o que o isolation level do
  DB está de fato fazendo por baixo de você. Prove a race num teste concorrente (Dor 2).
- **→ puxa a próxima:** travar uma linha é fácil; um checkout inteiro não é →
  transactions/sagas.

### Dor 9 — Checkout que falha no meio
- **Tópico:** transactions, consistência, atomicidade entre passos.
- **Fabricar:** um checkout que reserva stock E cria um order E chama payment — então
  mate-o entre os passos.
- **Observe:** stock reservado mas nenhum order, ou order sem payment — estado
  inconsistente.
- **Where:** Orders + Inventory + Payments.
- **Ideia de implementação:** primeiro faça errado (sem coordenação) e veja a bagunça.
  Aprenda onde termina a fronteira de uma transaction local de DB e por que ela não pode
  cobrir a chamada de payment. Esta é a semente do saga pattern.
- **→ puxa a próxima:** coordenar entre fronteiras → DDD para desenhar as fronteiras certo.

### Dor 10 — Modelar o domínio (DDD)
- **Tópico:** DDD, bounded contexts, aggregates.
- **Fabricar:** os domains do monólito estão emaranhados; uma mudança em Packing quebra
  Orders.
- **Observe:** o acoplamento — rastreie como uma mudança se propaga.
- **Where:** todos.
- **Ideia de implementação:** identifique aggregates e bounded contexts no papel; refatore
  os pacotes para que os contexts fiquem explícitos e independentes. Este é o trabalho de
  base que torna possível a separação de serviços mais tarde.
- **→ puxa a próxima:** as fronteiras entre domains estão limpas, mas o domínio ainda está
  colado na infraestrutura.

### Dor 11 — Domínio colado na infra (arquitetura hexagonal)
- **Tópico:** arquitetura hexagonal (ports & adapters), Clean Architecture, inversão de
  dependência.
- **Fabricar:** planeje a troca do MySQL por Postgres (Dor 12) e conte quantos arquivos *do
  domínio* precisariam mudar. Tente testar uma regra de Orders sem subir Spring nem banco.
- **Observe:** anotações JPA/Spring nas classes de domínio, regra de negócio espalhada em
  services colados no repository, testes lentos porque tudo precisa de contexto.
- **Where:** Orders (piloto), depois os demais.
- **Ideia de implementação:** refatore um context (Orders) para domínio puro (sem
  JPA/Spring) + ports (ex.: `OrderRepository` e `PaymentGateway` como interfaces do
  domínio) + adapters (JPA, REST). Meça antes/depois: arquivos tocados numa troca de infra,
  tempo da suíte de unit. Não converta tudo de uma vez — compare um context hexagonal com
  um que não é.
- **Aprofundar (quando não paga):** mais classes e mais mapeamento (modelo de domínio ↔
  entity JPA). Num CRUD simples (talvez o Catalog) pode não valer o custo — saiba
  argumentar onde vale.
- **→ puxa a próxima:** com ports & adapters, trocar o banco deveria ser trocar um adapter
  → prove.

### Dor 12 — Migração de banco (MySQL → Postgres, na mão) ⭐
- **Tópico:** migração de banco, o abismo entre teoria e realidade.
- **Fabricar:** você precisa de features que o MySQL não tem (JSONB indexável, partial
  indexes, DDL transacional) — e o mercado que você mira roda Postgres. Migre, com os
  milhões de linhas da Dor 5 dentro.
- **Observe:** o atrito real — diferenças de tipo, preocupações de downtime, integridade de
  dados, estratégia de cutover. E onde o adapter da Dor 11 *não* conteve a mudança (SQL
  nativo, tipos, sequences vs auto_increment).
- **Where:** cross-cutting.
- **Ideia de implementação:** faça **na mão**, de propósito. Esta dor só existe porque você
  começou no MySQL de propósito. Planeje a migração, execute, cuide dos dados. É a lição
  apontada como valiosíssima e muito comum para arquitetos de verdade. Introduza
  versionamento de schema (Flyway ou Liquibase) se ainda depende de `ddl-auto`.
- **→ puxa a próxima:** fronteiras limpas e banco definitivo convidam a primeira extração →
  microserviços.

---

## Fase 2 — Ficando distribuído (só agora)

### Dor 13 — Extrair o primeiro microserviço
- **Tópico:** microserviços, service boundaries, chamadas inter-serviço.
- **Fabricar:** Orders e Inventory precisam escalar/fazer deploy de forma independente.
- **Where:** Orders, Inventory.
- **Ideia de implementação:** recorte Inventory para o seu próprio serviço com os seus
  próprios dados. Sinta os novos custos imediatos: uma chamada de rede onde antes havia uma
  chamada de método, uma segunda coisa para fazer deploy, dados distribuídos. Não
  romantize — registre o que piorou. O docker-compose (Dor 3) passa a subir dois serviços.
- **→ puxa a próxima:** a chamada síncrona ingênua entre eles é uma armadilha.

### Dor 14 — Acoplamento síncrono derruba o sistema
- **Tópico:** os limites da comunicação síncrona.
- **Fabricar:** Orders chama Inventory por HTTP de forma síncrona. Derrube Inventory.
- **Observe:** Orders trava/falha também — o outage se propaga.
- **Where:** Orders → Inventory.
- **Ideia de implementação:** reproduza a falha em cascata. Este é o gancho emocional de
  tudo que é assíncrono a seguir. Meça como as threads se acumulam esperando.
- **Aprofundar (Virtual Threads):** aproveite para entender a alavanca do Java 21 — Virtual
  Threads (Project Loom). Onde no .NET você pensaria `async/await` para não prender thread,
  o Java te dá threads baratíssimas que podem bloquear sem custo. Meça o pool de threads
  empilhando com e sem Virtual Threads.
- **→ puxa a próxima:** desacople-os → messaging.

### Dor 15 — Messaging event-driven
- **Tópico:** messaging, arquitetura event-driven, brokers.
- **Fabricar:** substitua a chamada síncrona por eventos; processe "order placed" de forma
  assíncrona.
- **Observe:** Orders sobrevive quando Inventory está fora; os eventos esperam na queue.
- **Where:** Orders → Inventory / Notifications.
- **Ideia de implementação:** introduza Kafka ou RabbitMQ (escolha um, saiba por quê — as
  vagas pedem Kafka primeiro). Publique um evento na criação do order; consuma-o em
  Inventory e Notifications. Uma nova dor aparece na hora: ordenação, garantias de entrega,
  duplicatas.
- **→ puxa a próxima:** entrega "at least once" significa duplicatas → idempotency.

### Dor 16 — Idempotency & o problema do outbox
- **Tópico:** idempotency, efeitos exactly-once, transactional outbox.
- **Fabricar:** entregue o mesmo evento duas vezes; um order/parcel duplicado aparece.
- **Observe:** side-effects duplicados (dois labels, dois decrementos de stock).
- **Where:** Orders, Packing.
- **Ideia de implementação:** primeiro reproduza a duplicata. Então explore idempotency
  keys e o outbox pattern (por que escrever no DB e publicar um evento não podem ser dois
  passos separados). Isso liga direto à Dor 9.
- **→ puxa a próxima:** o sistema distribuído existe — mas só no seu notebook.

### Dor 17 — Só roda no meu PC (deploy na AWS)
- **Tópico:** cloud AWS, deploy contínuo (CD), serviços gerenciados.
- **Fabricar:** o Hermes só existe na sua máquina; ninguém (recrutador incluso) consegue
  acessá-lo, e cada "deploy" seria um ritual manual e frágil.
- **Observe:** quantos passos manuais existem e o que quebra entre "funciona no compose" e
  "funciona na nuvem" (rede, config, secrets, health checks).
- **Where:** cross-cutting (entrega).
- **Ideia de implementação:** imagens Docker (Dor 3) no ECR; serviços no ECS Fargate; banco
  no RDS Postgres; logs no CloudWatch; S3 para os labels (prepara as Dores 21–22). Estenda o
  pipeline da Dor 4 para fazer o deploy (CD). **Antes de tudo:** budget alarm na conta — uma
  conta AWS esquecida ligada é uma dor bem real.
- **Aprofundar (SQS/SNS vs Kafka):** crie um adapter SQS/SNS atrás do port de messaging
  (Dor 11) e compare com o Kafka (ou MSK): custo, ordenação, retenção, replay. Compare
  também Fargate vs EC2 vs EKS e saiba por que começou no mais simples.
- **→ puxa a próxima:** na nuvem, as coisas morrem de verdade → resiliência.

### Dor 18 — Resiliência (mate de propósito)
- **Tópico:** resiliência, circuit breakers, retries, timeouts, fallbacks.
- **Fabricar:** mate uma instância de Payments no meio de uma operação; faça a API falsa do
  carrier travar.
- **Observe:** como as falhas se espalham, como retries podem piorar (retry storms).
- **Where:** Payments, Shipping.
- **Ideia de implementação:** adicione timeouts primeiro (a correção mais barata), depois um
  circuit breaker, depois retries limitados com backoff. Prove cada um com uma falha que
  você injeta.
- **Aprofundar (backoff e fallback):** o retry cru amplifica o incêndio — use **exponential
  backoff** (esperas crescentes) para não formar retry storm. E entenda **fallback como
  degradação graciosa**: se o cálculo de frete cai, não devolva erro duro — devolva um valor
  padrão ou "frete temporariamente indisponível" e deixe o usuário seguir navegando. (Isto
  fecha o loop com a dúvida de "banco cai → degradar, não parar" levantada na spec 01.)
- **→ puxa a próxima:** o sistema aguenta falhas; agora vá para o coração do domínio →
  Packing.

---

## Fase 3 — Packing sob pressão ⭐ (o centro de gravidade do domínio)

### Dor 19 — Box selection (a otimização do packing)
- **Tópico:** algoritmos/otimização dentro de um serviço real, corretude vs performance.
- **Fabricar:** orders com muitos itens de dimensões variadas; escolha a menor box viável.
- **Observe:** a seleção ingênua desperdiça espaço ou não encaixa; meça o quão lenta uma
  busca brute-force fica quando a contagem de itens cresce.
- **Where:** Packing.
- **Ideia de implementação:** comece com uma heurística simples (first-fit por volume).
  Então sinta os limites dela e refine. Mantenha o algoritmo atrás de uma interface limpa
  para poder trocá-lo.
- **→ puxa a próxima:** agora faça isso milhares de vezes por hora → throughput.

### Dor 20 — Packing throughput
- **Tópico:** throughput, paralelismo, backpressure.
- **Fabricar:** simule muitas packing stations fazendo packing concorrentemente no pico de
  volume.
- **Observe:** onde engasga — contenção de DB, um lock compartilhado, uma chamada de label
  lenta.
- **Where:** Packing.
- **Ideia de implementação:** modele as stations como workers concorrentes; gere um workload
  de pico; ache o gargalo por medição, não por chute. Conserte aquele que os dados apontam.
  Considere Virtual Threads (Dor 14) para as stations concorrentes.
- **→ puxa a próxima:** e quando a chamada de label é lenta e externa? → offload assíncrono.

### Dor 21 — Dependência lenta: offload assíncrono
- **Tópico:** offload assíncrono de dependência lenta; request-reply vs fire-and-forget +
  notificação; por que polling não escala.
- **Fabricar:** o serviço (falso) de label demora minutos para responder (simule 3–4 min de
  latência). O request do usuário fica preso esperando.
- **Observe:** threads/conexões presas esperando; se você "resolve" com polling a cada X
  segundos, multiplique por milhares de usuários e veja o custo de rede/CPU explodir.
- **Where:** Packing → serviço de label; Notifications.
- **Ideia de implementação:** em vez de bloquear ou fazer polling, a API só **enfileira**
  ("label do parcel #123 pendente") e responde na hora "recebido, aviso quando pronto". Um
  **worker** consome a fila, chama a API lenta com calma, salva o resultado (S3, Dor 17) e
  dispara um evento; o usuário é **notificado** (push/e-mail/status via WebSocket). Liga-se
  à Dor 15 (messaging).
- **→ puxa a próxima:** repetir a chamada ao dar timeout gera label duplicado → idempotência.

### Dor 22 — Geração de label idempotente
- **Tópico:** idempotency numa integração, efeitos at-least-once vs exactly-once.
- **Fabricar:** repita um request de label após um timeout; dois labels são gerados.
- **Observe:** o label duplicado — um bug real e caro no fulfillment.
- **Where:** Packing → serviço de label (falso).
- **Ideia de implementação:** idempotency key por parcel; torne a chamada de label segura
  para repetir. Conecta diretamente à Dor 16.
- **→ puxa a próxima:** o conteúdo do packing precisa bater com o do picking → consistência
  cross-domain.

### Dor 23 — Consistência: packed = picked = ordered
- **Tópico:** consistência cross-service, reconciliation.
- **Fabricar:** force um mismatch — item com picking feito mas não com packing, ou packing a
  mais.
- **Observe:** a divergência entre as visões de três serviços sobre o mesmo order.
- **Where:** Orders, Warehouse, Packing.
- **Ideia de implementação:** desenhe uma checagem de reconciliation; decida como detectar e
  reparar drift. Eventual consistency tornada concreta.
- **→ puxa a próxima:** o volume continua subindo → carga.

---

## Fase 4 — Dados em escala

### Dor 24 — Carga & escalabilidade
- **Tópico:** escalabilidade, carga artificial, horizontal scaling.
- **Fabricar:** gere milhões de users/orders falsos e tráfego sustentado.
- **Observe:** a primeira coisa a cair sob carga real. Repare que auto-scaling tem delay —
  ele não responde a tempo do pico, e um monte de requests sofre no intervalo.
- **Where:** Catalog, Orders, Packing.
- **Ideia de implementação:** use um load generator; escale um serviço horizontalmente
  (mais tasks no ECS, Dor 17); descubra o que quebra quando você tem N instâncias (shared
  state, sticky sessions, conexões de DB).
- **Aprofundar (throttling / rate limiting):** escalar não é a única resposta. Proteja o
  downstream com **rate limiting / throttling** — limite requests por usuário/IP por segundo
  para não estourar o banco sob spike (cenário Black Friday). Sinta a diferença entre
  atrasar o problema (só escalar) e contê-lo (throttling + async).
- **→ puxa a próxima:** o primary do Postgres virou o teto → replication/partitioning.

### Dor 25 — Replication & partitioning
- **Tópico:** read replicas, sharding, partitioning.
- **Fabricar:** a carga de leitura satura o primary; uma table cresce além do confortável.
- **Observe:** replication lag; o trade-off de ler stale de uma replica.
- **Where:** cross-cutting.
- **Ideia de implementação:** adicione uma read replica e roteie leituras; então particione
  uma table grande (partitioning declarativo do Postgres). Sinta os trade-offs de
  consistência que você acabou de assinar.
- **Aprofundar (CAP / PACELC):** é aqui que os teoremas deixam de ser slide e viram
  concretos. Sob partição (CAP) você escolhe consistência ou disponibilidade; e mesmo sem
  partição (PACELC) você troca latência por consistência ao ler de uma replica. Nomeie qual
  ponto você escolheu e por quê.
- **→ puxa a próxima:** nem todo dado cabe bem em tabelas.

### Dor 26 — Schema rígido para dado heterogêneo (MongoDB)
- **Tópico:** banco de documentos, MongoDB, modelagem orientada ao padrão de acesso,
  polyglot persistence.
- **Fabricar:** o tracking do Shipping recebe eventos de carriers diferentes, cada um com
  payload próprio (scan no hub, tentativa de entrega com foto/geo, exceção na alfândega).
  Modele isso no relacional: EAV, dezenas de colunas nulas ou um blob JSON.
- **Observe:** o schema brigando com dado heterogêneo e append-only; a query "timeline de
  um parcel" virando joins e casts.
- **Where:** Shipping (tracking).
- **Ideia de implementação:** mova a timeline de tracking para o MongoDB (um documento por
  parcel ou por evento — decida pelo padrão de acesso), alimentada pelos eventos da Dor 15.
- **Aprofundar (JSONB basta?):** compare com JSONB no Postgres (Dor 12) — muitas vezes
  basta. Saiba argumentar quando um segundo datastore paga o custo de operar outro banco,
  sem joins nem transações fáceis entre collections.
- **→ puxa a próxima:** search no relacional para de escalar → search dedicada.

### Dor 27 — Search que escala (Elasticsearch)
- **Tópico:** search engines, indexing, manter dois datastores em sync.
- **Fabricar:** a full-text search de catálogo no DB relacional fica lenta e desajeitada.
- **Observe:** a complexidade e latência da query; então o problema de sync entre o DB e o
  index.
- **Where:** Catalog / Search.
- **Ideia de implementação:** mova a search para o Elasticsearch; mantenha em sync via os
  eventos que você já tem (liga de volta à Dor 15). O sync é a lição real, não a search em
  si.
- **→ puxa a próxima:** com tantas partes móveis, você está voando às cegas →
  observabilidade.

---

## Fase 5 — Enxergar o sistema

### Dor 28 — Observability & tracing
- **Tópico:** distributed tracing, metrics, structured logs.
- **Fabricar:** um único checkout agora cruza 5+ serviços e algo está lento, mas ninguém
  sabe onde.
- **Observe:** você literalmente não consegue dizer qual hop é o gargalo.
- **Where:** Observability (cross-cutting).
- **Ideia de implementação:** adicione correlation IDs, depois distributed tracing, depois
  metrics. A vitória é conseguir apontar o hop lento exato. (OpenTelemetry + Grafana, ou
  X-Ray/CloudWatch já que você está na AWS.)
- **Aprofundar (métricas que importam):** separe **business metrics** (orders/min, packs/hora)
  de **resource metrics** (CPU, memória, conexões). Defina um SLI de disponibilidade — ex.:
  % de requests não-5xx (1xx/2xx/3xx/4xx contam como "saudáveis", 5xx não) — e entenda o
  cálculo por trás de "99.9%". Saiba quais métricas disparam a decisão de scale in / scale
  out.
- **→ puxa a próxima:** agora você consegue ver a latência → vá caçá-la.

### Dor 29 — Caça à latência
- **Tópico:** análise de latência, tail latencies, otimização direcionada.
- **Fabricar:** o P99 do checkout está alto mesmo que as médias pareçam boas.
- **Observe:** a cauda — o 1% dos requests que são terríveis, e por quê.
- **Where:** Orders, Packing.
- **Ideia de implementação:** use o tracing da Dor 28 para achar a causa da cauda; conserte
  aquela uma coisa; prove que moveu o P99. Aprenda por que médias mentem.
- **→ puxa a próxima:** qualquer coisa nova que você quiser aprender — adicione uma dor e
  siga em frente.

---

## Dores parkeadas (dependem de pré-requisitos)

Estas já estão no radar, mas só se tornam acionáveis depois que o sistema tiver os
pré-requisitos abaixo. Não pule para elas antes disso.

### Dor 30 — Deploy sem downtime (canary / blue-green / rollback / feature toggle)
- **Tópico:** estratégias de release, deploy progressivo, rollback automático, feature
  toggles.
- **Pré-requisito:** múltiplos serviços/instâncias (Dor 13+), deploy na nuvem (Dor 17) e
  observabilidade com métricas de saúde (Dor 28) — o rollback é disparado por métrica.
- **Fabricar:** suba uma versão nova que degrada as métricas; veja o impacto se ela for para
  100% do tráfego de uma vez.
- **Where:** cross-cutting (entrega).
- **Ideia de implementação:** direcione só 1–5% do tráfego para a versão nova (**canary**);
  se as métricas de saúde degradam, **rollback automático** para a última versão estável;
  entenda também **blue-green** (ambiente antigo de pé enquanto o novo sobe; o load balancer
  aponta de volta na hora se o novo falhar). A primeira ação num deploy ruim é restaurar o
  serviço, não investigar o código.
- **Aprofundar (toggle e Kubernetes):** **feature toggle** separa *deploy* de *release* —
  o código sobe desligado e liga para uma fatia de usuários. E aqui o Kubernetes (EKS) pode
  enfim se justificar: rollouts progressivos nativos vs o que o ECS te dá.
- **→ puxa a próxima:** operar em produção puxa segurança, custo e mais observabilidade.

### Dor 31 — Retenção & expurgo de dados
- **Tópico:** data lifecycle, retention, archival, purga de dados antigos.
- **Pré-requisito:** volume de dados alto e sustentado (Dor 24+); idealmente já com
  partitioning (Dor 25).
- **Fabricar:** deixe orders/eventos antigos acumularem até tabelas e índices ficarem
  pesados; sinta queries e backups degradarem.
- **Where:** cross-cutting (dados).
- **Ideia de implementação:** defina critérios de retenção (o que guardar, por quanto tempo);
  arquive o frio (ex.: S3) e expurgue o que não precisa mais estar quente. Decida entre
  soft-delete, archival para storage barato e purge definitivo — e as implicações de cada um.
- **→ puxa a próxima:** conforme novas dores surgirem, adicione-as abaixo.

### Dor 32 — Infra clicada no console (Infrastructure as Code)
- **Tópico:** Infrastructure as Code, Terraform (ou CloudFormation), state, drift.
- **Pré-requisito:** o ambiente AWS da Dor 17 existindo e sendo usado.
- **Fabricar:** apague o ambiente e recrie do zero — ou crie um staging idêntico — clicando
  no console.
- **Observe:** passos esquecidos, configurações divergentes entre ambientes (drift), horas
  perdidas.
- **Where:** cross-cutting (infra).
- **Ideia de implementação:** descreva VPC, ECS, RDS, SQS e S3 em Terraform, com state
  remoto; o pipeline roda `plan` no PR e `apply` no merge. Prove recriando o ambiente
  inteiro com um comando.
- **→ puxa a próxima:** conforme novas dores surgirem, adicione-as abaixo.

---

## Adicionando as suas próprias dores

Quando você quiser estudar algo que não está listado aqui, não estude no vácuo. Adicione uma
linha neste espírito: qual é o tópico, que dor você vai fabricar, onde ela vive no Hermes, e
qual é uma ideia de implementação de partida. Então escreva a spec dela e vá.
