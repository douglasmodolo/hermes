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

## Fase 0 — Fundações (colocar o monólito para respirar)

### Dor 1 — Bootstrapping Spring Boot
- **Tópico:** estrutura de projeto Java + Spring Boot, a partir de uma mentalidade
  legado/.NET.
- **Fabricar:** nada ainda — suba um monólito com Catalog + Orders no MySQL, um endpoint
  que cria um order.
- **Where:** Catalog, Orders.
- **Ideia de implementação:** uma única app Spring Boot, package-by-domain (não
  por camada). Mapeie os conceitos .NET que você conhece (DI container, controllers, EF)
  para Spring (ApplicationContext, @RestController, JPA/Hibernate) — escreva esses
  mapeamentos no seu journal.
- **→ puxa a próxima:** agora você tem dados; faça-os doerem.

### Dor 2 — Queries lentas em escala (indexing)
- **Tópico:** performance de banco, indexes, query plans.
- **Fabricar:** insira 50–100M de products/orders falsos. Rode a search mais comum.
- **Observe:** veja o P99 subir de dezenas de ms para segundos; leia a saída do `EXPLAIN`.
- **Where:** Catalog.
- **Ideia de implementação:** escreva um script gerador de dados; meça uma query
  antes/depois de adicionar um index; aprenda a ler o plano de execução. Resista a adicionar
  um index às cegas — primeiro *veja* o full scan.
- **→ puxa a próxima:** indexes ajudam mas o DB ainda está quente → cache.

### Dor 3 — Caching & invalidation
- **Tópico:** caching, TTLs, invalidation, cache stampede.
- **Fabricar:** martele a query quente centenas de vezes/segundo; veja o CPU do DB
  disparar.
- **Observe:** compare a latência de hit vs miss; então mude o preço de um product e pegue
  stale reads.
- **Where:** Catalog.
- **Ideia de implementação:** introduza Redis como um read cache com um TTL. A lição real é
  a invalidation: o que acontece quando o dado subjacente muda? Crie de propósito um bug de
  stale-read e então conserte.
- **→ puxa a próxima:** agora você está fazendo malabarismo com consistência → transactions.

---

## Fase 1 — Integridade de order & concorrência

### Dor 4 — Overselling sob concorrência
- **Tópico:** concorrência, race conditions, estratégias de locking.
- **Fabricar:** dispare N compras simultâneas da última unidade em stock.
- **Observe:** o stock fica negativo — você vendeu mais do que tinha.
- **Where:** Inventory.
- **Ideia de implementação:** reproduza a race primeiro (read-then-write ingênuo). Então
  explore pessimistic vs optimistic locking e compare. Entenda o que o isolation level do
  DB está de fato fazendo por baixo de você.
- **→ puxa a próxima:** travar uma linha é fácil; um checkout inteiro não é →
  transactions/sagas.

### Dor 5 — Checkout que falha no meio
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

### Dor 6 — Modelar o domínio (DDD)
- **Tópico:** DDD, bounded contexts, aggregates.
- **Fabricar:** os domains do monólito estão emaranhados; uma mudança em Packing quebra
  Orders.
- **Observe:** o acoplamento — rastreie como uma mudança se propaga.
- **Where:** todos.
- **Ideia de implementação:** identifique aggregates e bounded contexts no papel; refatore
  os pacotes para que os contexts fiquem explícitos e independentes. Este é o trabalho de
  base que torna possível a separação de serviços mais tarde.
- **→ puxa a próxima:** fronteiras limpas convidam a primeira extração → microserviços.

---

## Fase 2 — Ficando distribuído (só agora)

### Dor 7 — Extrair o primeiro microserviço
- **Tópico:** microserviços, service boundaries, chamadas inter-serviço.
- **Fabricar:** Orders e Inventory precisam escalar/fazer deploy de forma independente.
- **Where:** Orders, Inventory.
- **Ideia de implementação:** recorte Inventory para o seu próprio serviço com os seus
  próprios dados. Sinta os novos custos imediatos: uma chamada de rede onde antes havia uma
  chamada de método, uma segunda coisa para fazer deploy, dados distribuídos. Não
  romantize — registre o que piorou.
- **→ puxa a próxima:** a chamada síncrona ingênua entre eles é uma armadilha.

### Dor 8 — Acoplamento síncrono derruba o sistema
- **Tópico:** os limites da comunicação síncrona.
- **Fabricar:** Orders chama Inventory por HTTP de forma síncrona. Derrube Inventory.
- **Observe:** Orders trava/falha também — o outage se propaga.
- **Where:** Orders → Inventory.
- **Ideia de implementação:** reproduza a falha em cascata. Este é o gancho emocional de
  tudo que é assíncrono a seguir. Meça como as threads se acumulam esperando.
- **→ puxa a próxima:** desacople-os → messaging.

### Dor 9 — Messaging event-driven
- **Tópico:** messaging, arquitetura event-driven, brokers.
- **Fabricar:** substitua a chamada síncrona por eventos; processe "order placed" de forma
  assíncrona.
- **Observe:** Orders sobrevive quando Inventory está fora; os eventos esperam na queue.
- **Where:** Orders → Inventory / Notifications.
- **Ideia de implementação:** introduza Kafka ou RabbitMQ (escolha um, saiba por quê).
  Publique um evento na criação do order; consuma-o em Inventory e Notifications. Uma nova
  dor aparece na hora: ordenação, garantias de entrega, duplicatas.
- **→ puxa a próxima:** entrega "at least once" significa duplicatas → idempotency.

### Dor 10 — Idempotency & o problema do outbox
- **Tópico:** idempotency, efeitos exactly-once, transactional outbox.
- **Fabricar:** entregue o mesmo evento duas vezes; um order/parcel duplicado aparece.
- **Observe:** side-effects duplicados (dois labels, dois decrementos de stock).
- **Where:** Orders, Packing.
- **Ideia de implementação:** primeiro reproduza a duplicata. Então explore idempotency
  keys e o outbox pattern (por que escrever no DB e publicar um evento não podem ser dois
  passos separados). Isso liga direto à Dor 5.
- **→ puxa a próxima:** coisas assíncronas falham silenciosamente → resiliência.

### Dor 11 — Resiliência (mate de propósito)
- **Tópico:** resiliência, circuit breakers, retries, timeouts, fallbacks.
- **Fabricar:** mate uma instância de Payments no meio de uma operação; faça a API falsa do
  carrier travar.
- **Observe:** como as falhas se espalham, como retries podem piorar (retry storms).
- **Where:** Payments, Shipping.
- **Ideia de implementação:** adicione timeouts primeiro (a correção mais barata), depois um
  circuit breaker, depois retries limitados com backoff. Prove cada um com uma falha que
  você injeta.
- **→ puxa a próxima:** agora empurre volume por ele → escalabilidade & carga.

---

## Fase 3 — Packing sob pressão ⭐ (o centro de gravidade do domínio)

### Dor 12 — Box selection (a otimização do packing)
- **Tópico:** algoritmos/otimização dentro de um serviço real, corretude vs performance.
- **Fabricar:** orders com muitos itens de dimensões variadas; escolha a menor box viável.
- **Observe:** a seleção ingênua desperdiça espaço ou não encaixa; meça o quão lenta uma
  busca brute-force fica quando a contagem de itens cresce.
- **Where:** Packing.
- **Ideia de implementação:** comece com uma heurística simples (first-fit por volume).
  Então sinta os limites dela e refine. Mantenha o algoritmo atrás de uma interface limpa
  para poder trocá-lo.
- **→ puxa a próxima:** agora faça isso milhares de vezes por hora → throughput.

### Dor 13 — Packing throughput
- **Tópico:** throughput, paralelismo, backpressure.
- **Fabricar:** simule muitas packing stations fazendo packing concorrentemente no pico de
  volume.
- **Observe:** onde engasga — contenção de DB, um lock compartilhado, uma chamada de label
  lenta.
- **Where:** Packing.
- **Ideia de implementação:** modele as stations como workers concorrentes; gere um workload
  de pico; ache o gargalo por medição, não por chute. Conserte aquele que os dados apontam.
- **→ puxa a próxima:** nunca imprimir um label duas vezes → labeling idempotente.

### Dor 14 — Geração de label idempotente
- **Tópico:** idempotency numa integração, efeitos at-least-once vs exactly-once.
- **Fabricar:** repita um request de label após um timeout; dois labels são gerados.
- **Observe:** o label duplicado — um bug real e caro no fulfillment.
- **Where:** Packing → serviço de label (falso).
- **Ideia de implementação:** idempotency key por parcel; torne a chamada de label segura
  para repetir. Conecta diretamente à Dor 10.
- **→ puxa a próxima:** o conteúdo do packing precisa bater com o do picking → consistência
  cross-domain.

### Dor 15 — Consistência: packed = picked = ordered
- **Tópico:** consistência cross-service, reconciliation.
- **Fabricar:** force um mismatch — item com picking feito mas não com packing, ou packing a
  mais.
- **Observe:** a divergência entre as visões de três serviços sobre o mesmo order.
- **Where:** Orders, Warehouse, Packing.
- **Ideia de implementação:** desenhe uma checagem de reconciliation; decida como detectar e
  reparar drift. Eventual consistency tornada concreta.
- **→ puxa a próxima:** o volume continua subindo → o banco não aguenta.

---

## Fase 4 — Dados em escala

### Dor 16 — Carga & escalabilidade
- **Tópico:** escalabilidade, carga artificial, horizontal scaling.
- **Fabricar:** gere milhões de users/orders falsos e tráfego sustentado.
- **Observe:** a primeira coisa a cair sob carga real.
- **Where:** Catalog, Orders, Packing.
- **Ideia de implementação:** use um load generator; escale um serviço horizontalmente;
  descubra o que quebra quando você tem N instâncias (shared state, sticky sessions,
  conexões de DB).
- **→ puxa a próxima:** o MySQL compartilhado agora é o teto → migre o DB.

### Dor 17 — Migração de banco (MySQL → Postgres, na mão) ⭐
- **Tópico:** migração de banco, o abismo entre teoria e realidade.
- **Fabricar:** o MySQL não dá conta / você precisa de features que ele não tem. Migre para
  Postgres.
- **Observe:** o atrito real — diferenças de tipo, preocupações de downtime, integridade de
  dados, estratégia de cutover.
- **Where:** cross-cutting.
- **Ideia de implementação:** faça **na mão**, de propósito. Esta dor só existe porque você
  começou no MySQL de propósito. Planeje a migração, execute, cuide dos dados. É a lição que
  o vídeo apontou como valiosíssima e muito comum para arquitetos de verdade.
- **→ puxa a próxima:** um DB ainda não é suficiente → replication/partitioning.

### Dor 18 — Replication & partitioning
- **Tópico:** read replicas, sharding, partitioning.
- **Fabricar:** a carga de leitura satura o primary; uma table cresce além do confortável.
- **Observe:** replication lag; o trade-off de ler stale de uma replica.
- **Where:** cross-cutting.
- **Ideia de implementação:** adicione uma read replica e roteie leituras; então particione
  uma table grande. Sinta os trade-offs de consistência que você acabou de assinar.
- **→ puxa a próxima:** search num DB relacional para de escalar → search dedicada.

### Dor 19 — Search que escala (Elasticsearch)
- **Tópico:** search engines, indexing, manter dois datastores em sync.
- **Fabricar:** a full-text search de catálogo no DB relacional fica lenta e desajeitada.
- **Observe:** a complexidade e latência da query; então o problema de sync entre o DB e o
  index.
- **Where:** Catalog / Search.
- **Ideia de implementação:** mova a search para o Elasticsearch; mantenha em sync via os
  eventos que você já tem (liga de volta à Dor 9). O sync é a lição real, não a search em
  si.
- **→ puxa a próxima:** com tantas partes móveis, você está voando às cegas →
  observabilidade.

---

## Fase 5 — Enxergar o sistema

### Dor 20 — Observability & tracing
- **Tópico:** distributed tracing, metrics, structured logs.
- **Fabricar:** um único checkout agora cruza 5+ serviços e algo está lento, mas ninguém
  sabe onde.
- **Observe:** você literalmente não consegue dizer qual hop é o gargalo.
- **Where:** Observability (cross-cutting).
- **Ideia de implementação:** adicione correlation IDs, depois distributed tracing, depois
  metrics. A vitória é conseguir apontar o hop lento exato.
- **→ puxa a próxima:** agora você consegue ver a latência → vá caçá-la.

### Dor 21 — Caça à latência
- **Tópico:** análise de latência, tail latencies, otimização direcionada.
- **Fabricar:** o P99 do checkout está alto mesmo que as médias pareçam boas.
- **Observe:** a cauda — o 1% dos requests que são terríveis, e por quê.
- **Where:** Orders, Packing.
- **Ideia de implementação:** use o tracing da Dor 20 para achar a causa da cauda; conserte
  aquela uma coisa; prove que moveu o P99. Aprenda por que médias mentem.
- **→ puxa a próxima:** qualquer coisa nova que você quiser aprender — adicione uma dor e
  siga em frente.

---

## Adicionando as suas próprias dores

Quando você quiser estudar algo que não está listado aqui, não estude no vácuo. Adicione uma
linha neste espírito: qual é o tópico, que dor você vai fabricar, onde ela vive no Hermes, e
qual é uma ideia de implementação de partida. Então escreva a spec dela e vá.
