# Hermes

**Português (pt-BR)** · [English](README.en.md)

> Um "projeto impossível": um clone, do zero, de um marketplace centrado em fulfillment,
> construído para aprender sistemas distribuídos sentindo a dor, não lendo sobre ela.

Hermes — mensageiro dos deuses, patrono das estradas e do comércio — cobre a **jornada
pós-compra inteira de um pedido**: checkout → armazém (picking, packing, dispatch) →
entrega / last-mile. A vitrine (catálogo, cart) existe apenas para alimentar pedidos e
levá-los até a parte que importa: **operações de fulfillment e delivery**.

Este é um laboratório de aprendizado pessoal. Ele nunca está "pronto". Ele cresce uma dor
por vez. O que move o projeto é a curiosidade sobre como sistemas de logística e
fulfillment funcionam por dentro, e o interesse em se aprofundar em sistemas distribuídos
na prática.

## Por que este projeto existe

Dois problemas matam a maioria dos estudos por conta própria:

1. **Obesidade mental** — acumular conhecimento sem propósito. Nunca fixa, nunca vira
   prática. O cérebro descarta o que não tem utilidade, contexto, emoção, repetição,
   resolução de problema ou sobrevivência atrelados.
2. **Falta de prática** — estudar certo mas nunca aplicar. Você esquece do mesmo jeito.

Hermes conserta os dois. Todo tópico que você estudar precisa (a) responder a uma *need*
real e (b) ser aplicado aqui, contra uma dor que você fabrica de propósito.

## Como trabalhar no Hermes (regras não negociáveis)

- **Você constrói tudo, do zero.** Estes docs são um mapa, não uma solução. Nenhum código
  é escrito pelo Claude a menos que você peça explicitamente.
- **Código, comentários, commits e linguagem de domínio em inglês.** Os docs e as specs
  são escritos em **português (pt-BR)** — mas os nomes de domínio e termos técnicos
  (Packing, Picking, SKU, saga, throughput, ...) permanecem em inglês, para ficarem
  consistentes com o código.
- **Spec-Driven Development.** Antes de escrever código para qualquer feature ou dor, você
  escreve uma spec primeiro (veja `docs/spec-driven-development.md`). A spec é onde você
  materializa a *need* antes da *solução*. Sem spec, sem código.
- **Uma dor por vez.** Escolha uma dor em `docs/pain-backlog.md`, escreva a spec dela,
  fabrique a dor, sinta-a, e então resolva. Registre em `docs/progress-journal.md`.
- **Deixe a arquitetura emergir.** Hermes começa como um **monólito** com **MySQL**. Ele
  não começa distribuído. Microserviços, novos bancos, message brokers — só entram quando
  uma dor exige. Começar deliberadamente no MySQL significa que você vai sentir depois a
  dor de migrar para Postgres, que é uma das lições mais valiosas que existem.

## Stack alvo

- **Linguagem / framework:** Java + Spring Boot.
- **Banco inicial:** MySQL (de propósito — a dor da migração é uma feature, não um bug).
- **Todo o resto** (Redis, Kafka/RabbitMQ, Postgres, Elasticsearch, containers, geradores
  de carga, tracing) entra **só quando uma dor exigir**. Nada é adicionado "porque é
  importante".

## Os documentos

| Arquivo | O que é |
|---|---|
| `docs/vision-and-scope.md` | O que é Hermes, a jornada pós-compra, os limites. |
| `docs/domains.md` | Os bounded contexts (Catalog, Orders, Warehouse, Packing, Shipping, ...). Onde cada dor vive. |
| `docs/pain-backlog.md` | **O coração.** A lista ordenada de dores a fabricar e bater, cada uma com uma ideia de implementação. |
| `docs/spec-driven-development.md` | Como fazer SDD direito — o template de spec e o workflow que você segue antes de codar. |
| `docs/progress-journal.md` | A régua da sua evolução. Uma entrada por sessão. |
| `docs/glossary.md` | Termos de fulfillment/logística em inglês, para a linguagem de domínio ficar consistente. |

## Primeiros passos sugeridos

1. `git init` nesta pasta e faça o primeiro commit com os docs.
2. Leia `docs/vision-and-scope.md` e `docs/domains.md` para segurar o quadro inteiro.
3. Abra `docs/pain-backlog.md`, escolha a **Dor #1** e leia `docs/spec-driven-development.md`.
4. Escreva a spec da Dor #1. Traga-a para revisão antes de escrever código.
