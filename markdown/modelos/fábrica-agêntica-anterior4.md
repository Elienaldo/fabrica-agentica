# Fábrica agêntica

Visão geral e regras da fábrica agêntica de software, no estado atual. Cada agente tem a sua ficha individual na pasta `modelos` (formato do Dr. Eggbot); este documento guarda o que é comum a todos: fluxo, regras, política de modelos, skills, ciclo de vida e pontos de atenção.

Dono do documento: Elienaldo Bittencourt. Redigido pelo Mestre de obras.

Legenda: itens marcados com **[proposta]** são sugestões minhas que o Elienaldo não pediu explicitamente. Confirme antes de tratar como regra.

---

## 1. Visão geral da fábrica agêntica

A fábrica recebe uma demanda e entrega código mergeado. Cada feature roda dentro de uma **sessão nomeada**, com a sua squad, criada e destruída pelo Dr. Eggbot.

### Fluxo, em fases

1. **Entrada:** fase sem diagrama próprio; descrita a partir das fichas do Supervisor e do Principal Product Engineer.
   - **Atores:** Supervisor (transversal e permanente), Dr. Eggbot e Principal Product Engineer; o humano só entra se faltar o projeto.
   - **Como acontece:** o Supervisor recebe a demanda (ticket, texto ou outra fonte). Só pergunta ao humano de qual projeto é a feature (nome ou caminho) se a demanda não trouxer. Cria a sessão nomeada em minúsculo, `feat/<nome-da-feature>` ou `fix/<nome-da-feature>`, e pede ao Dr. Eggbot o Principal Product Engineer, que sempre existe. Em seguida passa a feature ou a solicitação a ele. O Supervisor não classifica, não desenha, não implementa e não opina sobre o produto.
2. **Classificação:** é a porta de entrada do diagrama da fase research (`fase-research-v2`).
   - **Atores:** Principal Product Engineer, que atua como guardião do escopo.
   - **Como acontece:** o Principal Product Engineer recebe o pedido de feature e o classifica com 3 perguntas: muda arquitetura? Mexe em mais de um serviço? O requisito é ambíguo? Se alguma resposta for SIM, a feature vai para o research. Se todas forem NÃO, a feature é pequena: pula o research e entra no plan como solicitação direta. Nessa entrada ele também audita as skills do fluxo (the-judge, TLC Spec Lean, grill-me, wayfinder e prototype) e instala as que faltarem, sem pedir permissão; o wayfinder é a exceção e só se instala com aprovação do Elienaldo.
3. **Research** (só se a classificação mandar):
   - **Atores:** Principal Product Engineer e Product Engineer (produto, mesmo peso de voto), engenheiros de software A e B (modelos de famílias diferentes), design de front-end, advogado do diabo, roteirista e, em caso de impasse, o humano.
   - **Como acontece:** o time de research discute a feature e cada área toma uma posição. Os engenheiros de software A e B decidem o como técnico, cada um com a sua visão; os engenheiros de produto cuidam do escopo e da prioridade; o design de front-end desenha a experiência e decide o front. O advogado do diabo só levanta objeções enumeradas e justificadas, não vota e trava a fase enquanto houver objeção sem resposta. O time usa as skills grill-me (perguntas duras, em todo projeto), wayfinder (opcional, mapa para projeto grande ou do zero, só com ok do humano) e prototype (protótipo descartável do designer quando há dúvida visual ou de interação).
   - **Consenso:** as posições vão para o consenso por área (engenharia: como técnico; produto: escopo; design: front). Sem consenso, abre nova rodada, no máximo 2. Sem consenso depois de 2 rodadas, o humano decide.
   - **Saída:** com o consenso, ou com a decisão humana, o roteirista consolida as posições e escreve o **design doc** (problema, opções, decisão e trade-offs), que segue para a fase plan.
4. **Plan:**
   - **Atores:** Principal Product Engineer (entrega o insumo), Orquestrador de pipeline, modelo do Cursor (executa a skill TLC Spec Lean) e o humano, quando é avisado.
   - **Como acontece:** o Principal Product Engineer entrega ao orquestrador o design doc, vindo do research, ou a solicitação direta (texto, ticket do Notion ou do Jira, ou um ajuste), quando a feature pulou o research. O **Orquestrador de pipeline** solicita ao Cursor uma nova sessão com contexto limpo, passando todas as informações e o modelo (informado ou padrão), e pede ao modelo que dispare a skill **TLC Spec Lean** em modo plan. Regra: sempre pedir o planejamento E os checks; senão a skill só gera o `plan.md`. A skill gera os artefatos na pasta da feature: `plan.md` (o que o humano lê e contesta), `checks.md` (obrigações com prova, base do implement), `verification.md` (reservado, escrito pela própria skill na verificação, depois do build), `STATE.md` (decisões do projeto) e `LESSONS.md` (gerado a partir de falhas, nunca à mão).
   - **Saída:** com a pasta pronta, o orquestrador checa o resultado. Se falhou ou precisa de humano, avisa o humano, que decide. Se não, segue para o implement com o caminho da pasta da feature.
5. **Implement:**
   - **Atores:** Orquestrador de pipeline, modelo do Cursor (o menos custoso, executa a skill TLC Spec Lean) e o humano, que recebe a escalada.
   - **Como acontece:** o orquestrador de pipeline solicita ao Cursor uma nova sessão com contexto limpo e pede ao modelo que dispare a skill **TLC Spec Lean** em modo implement, passando o projeto, a feature atual e o caminho da pasta da feature (com `plan.md`, `checks.md`, `verification.md`, `STATE.md` e `LESSONS.md`); o modelo é opcional. Dentro da skill, tudo roda local: o build executa o que foi planejado, os testes unitários rodam a partir dos checks do plan e, depois da implementação, a skill executa a verificação, uma subtarefa interna dela, que confere se está tudo verde. Regra: teste vermelho é parada, e a asserção nunca é enfraquecida.
   - **Correção:** se ficar vermelho, abre uma rodada de correção e volta para os testes, até 3 rodadas. Se ainda estiver vermelho depois de 3 rodadas, escala ao humano.
   - **Saída:** com tudo verde, a skill faz o handoff final ao orquestrador com o caminho da pasta da feature. O orquestrador então solicita ao modelo do Cursor um push no GitHub; o push gera o PR automaticamente e encerra a fase implement.
6. **Review:**
   - **Atores:** Orquestrador de pipeline, modelo do Cursor (executa a skill the-judge), Curador amigável, Cirurgião e o humano.
   - **Como acontece:** com o PR aberto, o orquestrador de pipeline solicita ao Cursor uma nova sessão, antes de passar qualquer informação, e passa projeto, feature e número do PR. Em seguida solicita ao modelo (padrão ou específico) que dispare a skill **the-judge** para efetuar um **review do PR**, passando o número do PR (`--pr <número>`) e o modelo. A the-judge classifica os findings em blocker, should fix e nit, faz no máximo 3 rodadas por execução, e cada nova rodada revisa só o diff novo.
   - **Curador amigável:** pega o resultado do review da skill the-judge e o transforma em um HTML visual e amigável, com linguagem direta e simples para qualquer senioridade de engenheiro, exibido no app do Grok Bot. Destaca os blockers e os itens não bloqueantes e, principalmente, o que precisa de decisão humana, para que a decisão seja informada. Não corrige, não reclassifica e não muda a severidade. Se o review vier limpo, não gera HTML.
   - **Cirurgião:** corrige sozinho os itens de should fix e nit, mexendo só no necessário, e devolve o PR para a the-judge revisar de novo. Should fix complexo ou com risco vai para o humano. Quando há **blocker**, o cirurgião nunca corrige por conta própria: solicita intervenção humana, e só corrige o blocker com autorização do humano (que pode trazer uma sugestão de correção); nesse caso a the-judge valida e, se passar, o merge é automático, sem nova avaliação humana. **Limite de loops:** no máximo 3 rodadas de correção; se estourar as 3 rodadas, ele obrigatoriamente para, trava e chama intervenção humana.
   - **Travas anti-loop:** 3 rodadas com blocker param e escalam ao humano; should fix 3 vezes seguidas depois do cirurgião trava; blocker novo sempre vai para o humano; should fix complexo ou com risco vai para o humano.
7. **Merge:** fase sem diagrama próprio; descrita a partir do bloco "Merge automático" do diagrama `fase-pr`.
   - **Atores:** Orquestrador de pipeline e, antes dele, a the-judge, que dá o veredito.
   - **Como acontece:** quando a última rodada da the-judge vem limpa e não há blocker pendente (nit pode ficar), o orquestrador de pipeline dispara o merge no GitHub. O PR mergeado é o handoff final da fábrica (ver ponto de atenção sobre isso na seção 6).
8. **Aviso:** o Supervisor avisa o Elienaldo no merge, pelo app do Grok.

```mermaid
flowchart LR
  SU[Supervisor] --> EG[Dr. Eggbot cria o Principal Product Engineer]
  EG --> PE{Principal Product Engineer: 3 perguntas}
  PE -- alguma SIM --> RS[Research: time, advogado, roteirista]
  PE -- todas NÃO --> PL
  RS -- design doc --> PL[Plan: orquestrador de pipeline + TLC Spec Lean]
  PL -- plan.md + checks.md --> IM[Implement: orquestrador de pipeline + TLC Spec Lean]
  IM -- push no GitHub cria o PR --> JU[Review: the-judge]
  JU -- findings --> CU[Curador amigável]
  JU -- should fix / nit --> CI[Cirurgião]
  CI -- nova rodada --> JU
  JU -- limpo --> OP[Orquestrador de pipeline dispara o merge]
  OP --> SU
  SU -. avisa no merge .-> EL((Elienaldo))
```

### Quem é quem

| Agente | Papel | Ficha |
| --- | --- | --- |
| Supervisor | Recebe a demanda, pergunta o projeto só se ela não trouxer, cria a sessão `feat/` ou `fix/`, pede ao Dr. Eggbot o Principal Product Engineer e coordena do pedido ao merge; transversal e permanente; avisa o Elienaldo no merge pelo app do Grok e, depois do ok final, pede ao Dr. Eggbot a destruição da sessão. | `supervisor.md` |
| Dr. Eggbot | Cria e destrói os bots e os chats da sessão. | ainda sem ficha no formato novo |
| Principal Product Engineer | Entrada da feature na squad, guardião do escopo e engenheiro mais experiente; classifica a feature com 3 perguntas; audita as skills do fluxo e instala as que faltarem; mesmo peso de voto do Product Engineer. | `principal-product-engineer.md` |
| Product Engineer | Segundo engenheiro de produto, com o mesmo peso de voto do Principal Product Engineer; instala a skill do fluxo que faltar. | `product-engineer.md` |
| Engenheiro de software A e B | Decidem o como técnico, com modelos de famílias diferentes. | ainda sem ficha no formato novo |
| Design de front-end | Especialista em front-end: decide a experiência (UX e UI), cria o protótipo descartável quando há dúvida visual ou de interação e fecha o consenso da área de design. | `design-front-end.md` |
| Advogado do diabo | Levanta objeções enumeradas e justificadas; não vota, não decide e não propõe solução; trava a fase enquanto houver objeção sem resposta. | `advogado-do-diabo.md` |
| Roteirista | Consolida as posições e escreve o design doc; não decide e não vota. | ainda sem ficha no formato novo (só existe como seção 4.8 de `modelos-agentes-anterior.md`) |
| Orquestrador de pipeline | Atua em plan, implement e review: solicita ao Cursor uma sessão nova com contexto limpo por fase; pede ao modelo que dispare a TLC Spec Lean (plan e implement) e a the-judge (review do PR, com número do PR e modelo); pede o push que cria o PR; dispara o merge. | `orquestrador-de-pipeline.md` |
| Cirurgião | Corrige should fix e nit do PR e devolve à the-judge; escala should fix complexo; blocker só com autorização humana; máximo de 3 rodadas de correção, depois trava e escala ao humano. | `cirurgiao.md` |
| Curador amigável | Transforma os findings da the-judge em HTML amigável no app do Grok Bot, destacando o que precisa de decisão humana; não corrige nem muda severidade. | `curador-amigavel.md` |

---

## 2. Regras gerais

### 2.1 Sessão nomeada e ID por feature (paralelismo)

- Cada feature tem uma sessão cujo nome é o prefixo em minúsculo mais o nome da feature:
  - `feat/<nome-da-feature>` para feature nova.
  - `fix/<nome-da-feature>` para correção.
- O nome da feature em formato slug (minúsculas, hifens) é o **ID da feature**. Tudo da feature leva esse ID: sessão, pasta, branch, PR, nome dos agentes da squad.
- O ID separa o trabalho de features diferentes, então várias features rodam **em paralelo**, cada uma com a sua squad, sem compartilhar estado.
- **[proposta]** Se o ID já existir ativo, o Dr. Eggbot recusa a criação e devolve ao Supervisor, em vez de reaproveitar a squad.
- **[proposta]** Cada feature tem uma pasta própria; o caminho dela viaja junto com o design doc no handoff.

### 2.2 Sessões das fases são pedidas ao Cursor

A abertura de sessão de cada fase (plan, implement, review) é sempre solicitada ao **Cursor**, nunca ao modelo. O modelo entra como parâmetro da solicitação e só roda skills e executa ações.

### 2.3 Consenso por área

No research, cada área decide o que é seu:

| Área | Decide |
| --- | --- |
| Engenharia (os dois engenheiros de software) | o como técnico |
| Produto (Principal Product Engineer e Product Engineer, mesmo peso de voto) | escopo e prioridade |
| Design (front-end) | o front |

O consenso é por área, não por maioria geral. A prioridade única dos engenheiros de produto é entregar valor ao usuário. **[proposta]** Sem consenso depois de 2 rodadas, o humano desempata.

### 2.4 Advogado do diabo sem voto

O advogado do diabo **não vota**. Só levanta objeções, enumeradas e justificadas. A fase **não avança** enquanto houver objeção sem resposta registrada (ele trava a fase).

### 2.5 Design doc como saída padrão do research

A saída do research é sempre o **design doc** (problema, opções, decisão e trade-offs), escrito pelo roteirista. Ele é o que o plan recebe.

### 2.6 Classificação e atalho

O Principal Product Engineer faz 3 perguntas: a feature **muda arquitetura**? **Mexe em mais de um serviço**? Tem **ambiguidade de requisito**? Qualquer SIM manda para o research. Todas NÃO vão pelo atalho direto ao plan, sem research.

### 2.7 Skills que faltam

O Principal Product Engineer audita as skills do fluxo (the-judge, TLC Spec Lean, grill-me, wayfinder e prototype). Se faltar alguma, ele ou o Product Engineer avisam e instalam sem pedir permissão ao humano, porque a fábrica é automatizada e não pode parar por falta de skill. Exceção: o wayfinder só se instala com aprovação do Elienaldo.

### 2.8 Limites de rodadas

- Implement: até 3 rodadas de correção na verificação; depois escala ao humano.
- the-judge: máximo de 3 rodadas de revisão por execução.
- Cirurgião: máximo de 3 rodadas de correção; depois trava e escala ao humano.

### 2.9 Regras para o humano

- **Blocker é sempre humano.** O cirurgião não corrige blocker sozinho; se o humano não intervir, o fluxo trava e escala.
- Should fix complexo escala ao humano.
- O humano pode, opcionalmente, direcionar a forma de correção. Não é obrigatório.

---

## 3. Política de modelos de LLM

| Fase | Faixa de modelo | Por quê |
| --- | --- | --- |
| Research | **robustos** | decisões de arquitetura e design; erro aqui custa caro depois |
| Plan | **medianos** | transformar decisão em plano e checks; trabalho estruturado |
| Implement | **mais baratos** | a spec e os gates já limitam o trabalho; custo por volume |

Este documento fala em **faixas** (robusto, mediano, barato). O modelo concreto de cada faixa é escolha do Elienaldo e do Dr. Eggbot na hora de criar, conforme o que estiver disponível e o preço.

Regra dos dois engenheiros de software: **modelos de famílias diferentes**, para que a discussão não seja a mesma opinião duas vezes.

| Agente | Faixa |
| --- | --- |
| Supervisor | mediano **[proposta]** |
| Dr. Eggbot | mediano **[proposta]** |
| Principal Product Engineer | mediano **[proposta]** |
| Product Engineer | mediano **[proposta]** |
| Engenheiro de software A | robusto |
| Engenheiro de software B | robusto, família diferente da A |
| Design de front-end | robusto |
| Advogado do diabo | robusto |
| Roteirista | robusto |
| Orquestrador de pipeline | modelo do Cursor: mediano em plan, mais barato em implement |
| Cirurgião | barato **[proposta]**, escala o que for complexo |
| Curador amigável | a definir (a faixa foi retirada da ficha) |
| the-judge (skill) | mediano **[proposta]**, preferir família diferente da que implementou |

---

## 4. Skills

Skills não são agentes: são instruções que um agente carrega. Não há criação ou destruição de skill por feature.

### 4.1 the-judge

- **O que é:** skill de **review de PR**.
- **Quem usa:** o modelo, solicitado pelo orquestrador de pipeline na fase de review, em sessão nova do Cursor.
- **Como funciona:** até **3 rodadas** por execução; classifica cada finding em **blocker**, **should fix** ou **nit**. Nova rodada, depois de uma correção, só revisa o diff novo.
- **Saída:** findings, entregues ao curador amigável e ao cirurgião.
- **Observação:** o texto original da skill vem do Elienaldo; este documento só a descreve.

### 4.2 TLC Spec Lean

- **O que é:** skill de especificação e execução (nome real: `tlc-spec-lean`).
- **Usos na fábrica:**
  - **Plan:** gera na pasta da feature `plan.md` e `checks.md` (sempre pedir planejamento E checks), além de `verification.md` (reservado), `STATE.md` e `LESSONS.md`.
  - **Implement:** executa por fases, com **commit por fase** e gates de verificação.
- **Quem usa:** o modelo do Cursor, disparado pelo orquestrador de pipeline, com o modelo da fase.

### 4.3 grill-me

- **O que é:** skill de **questionamentos**.
- **Quem usa:** o time de research, em todo projeto.

### 4.4 wayfinder

- **O que é:** skill **opcional**, para projeto **do zero** ou grande e complexo. Substitui o grill-me nesses casos.
- **Instalação:** só se o Elienaldo aprovar.

### 4.5 prototype

- **O que é:** skill do **designer** para **protótipo descartável**, que nunca vira código de produção.
- **Quando usa:** dúvida visual ou de interação: feature nova, mudança grande de front-end ou pedido do humano.

---

## 5. Ciclo de vida da squad

| Momento | Quem faz | O quê |
| --- | --- | --- |
| Demanda chega | Supervisor | Recebe a demanda, abre a sessão `feat/<feature>` ou `fix/<feature>` e pede ao Dr. Eggbot o Principal Product Engineer. |
| Criação | Dr. Eggbot | Cria os agentes com o ID da feature. O Principal Product Engineer, o orquestrador de pipeline, o cirurgião e o curador amigável existem em toda sessão; o time de research só quando a classificação mandar. |
| Execução | Orquestrador de pipeline e squad | O orquestrador atua só dentro das fases plan, implement e review: solicita ao Cursor a abertura das sessões e dispara as skills. O Supervisor continua transversal e permanente. |
| Merge | Orquestrador de pipeline | Dispara o merge no GitHub quando a última rodada da the-judge vem limpa e não há blocker pendente. |
| Aviso | Supervisor | Avisa o Elienaldo no merge. |
| Destruição | Dr. Eggbot, a pedido do Supervisor | Depois do ok do humano sobre a finalização, destrói os bots e os chats exclusivos da sessão. Supervisor e Dr. Eggbot continuam. |
| Cancelamento | Supervisor, a pedido do Elienaldo **[proposta]** | Dr. Eggbot destrói a sessão sem merge. |

Pontos que o Dr. Eggbot deve receber como decisão:

- **Criar tudo de uma vez ou por fase?** Se a feature pega o atalho, os agentes de research não são usados. Sugestão **[proposta]**: criar os agentes de research só quando o Principal Product Engineer classificar como SIM.
- **Destruir quando?** Já decidido: depois do ok do humano sobre a finalização.

---

## 6. Pontos de atenção que o Mestre de obras vê

- **Aviso ao Elienaldo com dono único.** Quem avisa é o Supervisor, pelo app do Grok. O orquestrador de pipeline não manda aviso, para não haver mensagem duplicada. O aviso deve ter tom claro e direto: o que aconteceu e se há decisão pendente. Interpretei "mediador", do pedido, como o Supervisor; me diga se era outro agente.
- **Merge sem descrição na ficha.** Foi decidido que o orquestrador de pipeline dispara o merge, mas a ficha dele não descreve esse passo nem a condição (última rodada da the-judge limpa e sem blocker pendente; nit pode ficar). Falta escrever isso, ou o merge fica sem dono.
- **Contadores de rodadas sem dono.** A proposta de o Supervisor guardar os contadores de rodadas saiu da ficha. Hoje ninguém guarda; o `STATE.md` da TLC Spec Lean não tem contador de rodadas.
- **Humano como saída de vários lugares** (desempate de consenso, should fix complexo, blocker, 3 rodadas estouradas). Sem prazo de resposta definido, a sessão fica parada. Defina o que fazer quando o humano não responde.
- **Merge depende só do veredito da the-judge.** Vale um gate de CI como segunda condição.
- **Diagramas desatualizados.** `fase-pr` e `fase-implement` (e os `.md` das fases) ainda mostram componentes que não existem mais. Precisam ser refeitos.
- **Fichas pendentes.** Faltam as fichas no formato novo do Dr. Eggbot, dos engenheiros de software A e B e do roteirista (este só existe como seção 4.8 de `modelos-agentes-anterior.md`), e `principal-product-engineer.md` ainda cita o "orquestrador" no Acordar, que agora é o Supervisor.
