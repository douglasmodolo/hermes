# Prompt: analisar o repo antigo e enriquecer a documentação do Hermes

Cole o texto abaixo no **Claude Code**, rodando dentro da pasta do Hermes
(`C:\repos\hermes`), com a skill `estudar` instalada. Substitua `C:\repos\fleettrack`
pelo caminho real do repositório anterior.

> **Importante:** este prompt manda o Claude Code **só documentar** — analisar o repo
> antigo e aprimorar os `.md` do Hermes. Ele **não** deve copiar código do repo antigo pra
> dentro do Hermes nem implementar nada. O Hermes é construído do zero, à mão.

---

## Prompt para colar

```
Contexto: estou construindo um projeto chamado Hermes — um marketplace focado em
fulfillment/packing — como laboratório de estudo de sistemas distribuídos (Java + Spring
Boot). A metodologia está nos docs desta pasta (docs/pain-backlog.md, docs/domains.md,
docs/vision-and-scope.md, docs/spec-driven-development.md, docs/progress-journal.md,
docs/glossary.md). Leia todos eles primeiro para entender o método e o projeto.

Regras que você DEVE respeitar:
- NÃO implemente nada. Não escreva código de aplicação. Seu trabalho aqui é só ANALISAR e
  DOCUMENTAR.
- NÃO copie código do repo antigo para dentro do Hermes. O Hermes é construído do zero, à
  mão, por mim. O repo antigo serve apenas como fonte de APRENDIZADO.
- Trabalhe em português comigo na conversa. Os documentos e specs do Hermes são escritos
  em português (pt-BR), mas os nomes de domínio e termos técnicos (Packing, saga, SKU, ...)
  permanecem em inglês, para ficarem consistentes com o código.

Tarefa: analise o repositório antigo em <CAMINHO_DO_REPO_ANTIGO>. Ele foi feito antes,
sem método, atropelando etapas e com implementação automática — então é um bom retrato de
"o que eu já tentei" e "onde furei". A partir dessa análise, ENRIQUEÇA a documentação do
Hermes. Especificamente:

1. Faça um inventário do repo antigo (sem despejar arquivos inteiros): que domínios/módulos
   existem, que tecnologias foram usadas (banco, mensageria, cache, etc.), que arquitetura
   emergiu (monolito? serviços?), e o estado geral (o que funciona, o que está pela metade).

2. Cruze esse inventário com docs/domains.md e docs/pain-backlog.md do Hermes:
   - Aponte quais dores do backlog o repo antigo JÁ TOCOU (mesmo que mal), e como.
   - Aponte quais etapas foram ATROPELADAS — ex.: já tem microserviço mas nunca passou pela
     dor da chamada síncrona que derruba o sistema; já usa Postgres sem ter sentido a dor da
     migração; já tem cache sem ter sentido a dor da query lenta. Liste esses "pulos".
   - Para cada etapa atropelada, descreva como VOLTAR nela do jeito certo no Hermes: qual
     dor fabricar primeiro para que aquele conhecimento realmente fixe.

3. Extraia dores REAIS que apareceram no repo antigo (bugs, gambiarras, coisas que
   quebraram, decisões que se provaram ruins). Cada uma dessas é ouro — vira uma dor
   específica e contextualizada. Proponha adicioná-las ao docs/pain-backlog.md, no formato
   já usado lá (topic / manufacture / where / implementation idea / → pulls next), com
   numeração seguindo a que já existe.

4. Se o domínio do repo antigo revelar partes de fulfillment/packing que meus docs ainda
   não cobrem bem, sugira melhorias em docs/domains.md e docs/vision-and-scope.md.

5. Atualize docs/glossary.md com termos de domínio que apareçam no repo antigo e ainda não
   estejam lá.

Entregue primeiro um RESUMO na conversa (o inventário + os pulos + as dores novas
propostas) para eu revisar. Só depois de eu aprovar, edite os arquivos .md. Não edite nada
antes da minha aprovação.

Ao editar, preserve o estilo e a estrutura dos documentos existentes. Marque claramente o
que veio da análise do repo antigo (ex.: uma seção "Learnings from the previous attempt"
ou uma coluna/nota indicando a origem), para eu saber o que é herança e o que é novo.
```

---

## Depois que ele terminar

- Revise o resumo antes de deixar ele editar os arquivos.
- Faça um commit separado só com o enriquecimento da doc (ex.:
  `docs: enrich pain backlog with learnings from previous attempt`) pra fica rastreável no
  git history — que é a sua régua de evolução.
- Quando quiser começar a estudar de fato, é só invocar `/estudar` e escolher uma dor.
```
