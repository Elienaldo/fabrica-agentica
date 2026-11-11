# Fábrica Agêntica

Fábrica de desenvolvimento de software feita de bots: recebe uma demanda e entrega código mergeado, passando por fases com verificação e com o humano só onde a decisão é dele.

**Dono:** Elienaldo Bittencourt · **Última atualização deste README:** 2026-11-11

**Caminho do projeto:** `C:\Users\Elienaldo\Projetos\Fabrica-Agentica`  
**Scripts operacionais (fora deste repositório):** `C:\Users\Elienaldo\GrokBot-Scripts`

> **Estado atual:** o caminho rápido (Dev Expresso, Supervisor, Cursor e volta) foi validado de ponta a ponta no teste real `req-20261103-001`. O fluxo padrão (research, plan, implement, review) está desenhado e documentado. O script local de **criação** de seção existe e é o procedimento operacional do Supervisor. O script local de **destruição** permanece bloqueado (`validated=false` em `destruir-secao-config.json`). O teste real de 08/11/2026 (America/Bahia) passou no inventário do Cirurgião da seção `Teste` e parou na identificação do menu contextual; nada foi excluído. Candidatos de correção ficaram só em staging; a configuração de produção não foi liberada.

---

## Sumário

1. [O que é](#1-o-que-é)
2. [Objetivo](#2-objetivo)
3. [Função de cada parte](#3-função-de-cada-parte)
4. [Fases do fluxo](#4-fases-do-fluxo)
5. [Comunicação bidirecional com o Cursor](#5-comunicação-bidirecional-com-o-cursor)
6. [Pedido órfão](#6-pedido-órfão-seção-9-da-supervisor-regras)
7. [Seções por feature](#7-seções-por-feature)
8. [Padrões adotados](#8-padrões-adotados)
9. [Como utilizar](#9-como-utilizar)
10. [Estrutura de pastas](#10-estrutura-de-pastas)
11. [O que cada documento cobre](#11-o-que-cada-documento-cobre)
12. [Pendências e pontos em aberto](#12-pendências-e-pontos-em-aberto)

---

## 1. O que é

Um conjunto de bots do Grok Bot que trabalham em equipe com agentes de nuvem do Cursor. Os bots coordenam, decidem e controlam o fluxo. Quem escreve o código é o agente do Cursor, que roda numa VM própria, em branch própria. Cada feature roda numa **seção** da barra lateral, com a sua squad. A **criação** e a **destruição** da seção são iniciadas pelo Supervisor, via script local no PC, nas condições das seções 23 e 24 de `skills/supervisor-regras.md`. O Dr. Eggbot cria bots e equipes; **não participa da destruição**.

## 2. Objetivo

- Transformar uma demanda (ticket, texto, ajuste pequeno) em PR mergeado, com plano, testes e review no meio do caminho.
- Ter um só ponto de contato com o Cursor (o Supervisor) e um só canal com o humano (também o Supervisor), para tudo ficar rastreado.
- Fazer cada fase terminar com uma verificação e com critério de pronto, sem handoff sem dono nem loop sem saída.
- Manter o humano no comando: ele decide quando há impasse, blocker, risco, validação de protótipo ou merge fora do padrão.

## 3. Função de cada parte

### Peças de comunicação com o Cursor

| Parte | O que é | Função |
| --- | --- | --- |
| **Supervisor** | Bot permanente | Ponte única entre a fábrica e o Cursor, e único canal com o humano. Resolve o workspace, lança os agentes, mantém o livro-razão, casa o retorno com o pedido, avisa o humano. Inicia a criação e a destruição local da seção, quando autorizadas e, no caso da destruição, com capacidades validadas. Não classifica, não desenha, não implementa, não revisa. Regras gerais na skill `supervisor-regras` (seções 1 a 27). |
| **Dev Expresso** | Bot permanente | Caminho à parte para desenvolvimentos pequenos (fix, label, cor de botão), fora do fluxo padrão. Recebe do humano, repassa ao Supervisor, devolve ao humano o retorno completo (resumo, prints, arquivos, testes, links). Aplica o freio de escopo: pedido grande vai ao fluxo padrão. Não tem skill própria. |
| **Cursor** | Agentes de nuvem, fora do Grok Bot | Executam o trabalho de código numa VM, em branch nova, a pedido do Supervisor. Não entram em grupo de chat. |
| **Livro-razão** | Arquivo de estado do Supervisor | Registra cada pedido (`request_id`), o ID do agente (`bc-…`), grupo, feature, fase e a entrega (commit, PR). É a base do casamento do retorno. |
| **Varredura de status** | Rotina do Supervisor, a cada cerca de 10 minutos | Rede de segurança: consulta no Cursor, por ID de agente e por run, cada pedido aberto. Sem pedido aberto, sai em silêncio. |
| **Receptora de webhook do Cursor** | Inativa, fora do fluxo | Rotina desenhada e nunca executada, sem uso previsto por enquanto (futuro/opcional). O retorno do Cursor vem pela notificação de conclusão (seção 5). Documento `automacao-receptora-cursor.md`. |

### Squad do fluxo padrão

| Bot | Função |
| --- | --- |
| **Dr. Eggbot** | Cria a equipe e os grupos da feature, depois da seção vazia existir. Não inicia nem executa a destruição. |
| **Principal Product Engineer** | Guardião do escopo: classifica a demanda, roteia as skills e orquestra o research. |
| **Product Engineer** | No research, traz a visão técnica e de produto, com o mesmo peso de voto do Principal. |
| **Design de front-end** | Desenha a experiência do front no research e o protótipo descartável, quando houver dúvida visual. |
| **Advogado do diabo** | Levanta objeções justificadas no research. Não vota; trava a fase enquanto houver objeção sem resposta. |
| **Orquestrador de pipeline** | Conduz plan, implement e review: recebe os insumos do Supervisor (nunca direto do Principal), devolve a ele o pedido ao Cursor e faz o merge. |
| **Curador amigável** | Transforma o resultado do review em HTML visual e simples para o humano decidir. |
| **Cirurgião** | Corrige should fix e nit. Blocker só com autorização do humano. |
| **Âncora** | Bot permanente de apoio à UI. O script de criação usa a Âncora para abrir a seção nova e a devolve à seção Fábrica Agêntica. Não faz parte da squad da feature. É protegida contra destruição. |
| **Mestre de obras** | Desenha, documenta, configura e testa a própria fábrica. Não faz parte do fluxo de uma feature. |

### Skills usadas pelos agentes do Cursor

`TLC Spec Lean` (`tlc-spec-lean`, plan e implement; o **Verifier** é subagente interno dela, não é fase nem bot separado), `the-judge` (review do PR), `tlc-prd` e `tlc-discover` (saída do research), `wayfinder`, `grill-me` e `prototype` (opcionais). O detalhe de cada uma está em `markdown/modelos/fábrica-agêntica.md`, seção 4.

## 4. Fases do fluxo

```mermaid
flowchart LR
  E[1. Entrada] --> C{2. Classificação}
  C -- alguma SIM --> R[3. Research]
  C -- nenhuma SIM --> P
  R -- design doc ou PRD --> P[4. Plan]
  P -- pasta da feature --> I[5. Implement]
  I -- push cria o PR --> V[6. Review]
  V -- limpo, sem blocker --> M[7. Merge]
  M --> A[8. Aviso ao humano]
```

| Fase | Quem faz | O que acontece | O que sai |
| --- | --- | --- | --- |
| 1. Entrada | Supervisor, Dr. Eggbot | O Supervisor recebe a demanda, pergunta o projeto só se faltar, cria a seção da feature pelo script local (seção 7) e só então pede os bots ao Dr. Eggbot. Abre a branch e pede o Principal Product Engineer. | feature e seção |
| 2. Classificação | Principal Product Engineer | Na visão atual (`fábrica-agêntica.md`): 5 perguntas (arquitetura; mais de um serviço ou componente compartilhado; requisito ambíguo; difícil de desfazer; autenticação, dados pessoais ou segredos). Nenhum SIM é simples e vai direto ao plan; um ou dois SIM é média; três ou mais SIM, ou produto novo, é alta. SIM nas perguntas 4 ou 5 dá no mínimo média. O humano pode fixar o nível com uma tag. Ver pendência sobre textos que ainda citam 3 perguntas. | rota |
| 3. Research (a fase de busca) | Principal, Product Engineer, design de front-end, advogado do diabo | Decisão em consenso, no máximo 2 rodadas; sem consenso, o humano decide. Entrevista obrigatória, quando houver, interrompe o avanço. O OK humano do protótipo é obrigatório antes do plan (seção 22 de `supervisor-regras`). | PRD (média) ou design doc (alta) |
| 4. Plan | Orquestrador, Cursor com TLC Spec Lean | Sessão nova de contexto limpo; sempre pede plano e checks. Só começa depois do OK do protótipo, quando houve research com protótipo. | `plan.md`, `checks.md`, `verification.md`, `STATE.md`, `LESSONS.md` |
| 5. Implement | Orquestrador, Cursor com TLC Spec Lean | Build, testes e verificação dentro da skill (Verifier interno). Teste vermelho é parada, e a asserção nunca é enfraquecida. No máximo 3 rodadas de correção; depois, escala ao humano. | PR |
| 6. Review | Orquestrador, Cursor com the-judge, Curador, Cirurgião | A the-judge classifica em blocker, should fix e nit (máx. 3 rodadas por execução). O Cirurgião corrige should fix e nit (máx. 3 rodadas). Blocker e should fix complexo vão ao humano. | findings, HTML |
| 7. Merge | Orquestrador | Merge no GitHub quando a última rodada da the-judge vem limpa e sem blocker pendente (nit pode ficar). | PR mergeado |
| 8. Aviso | Supervisor | Avisa o humano. A destruição só pode ser iniciada pelo Supervisor via script local, com autorização explícita de alvo e alcance e com capacidades validadas, conforme a seção 24 de `supervisor-regras`. O merge, por si só, não autoriza destruir. Hoje o script permanece bloqueado. | aviso |

**Sessão nova em toda fase:** Toda fase nova (research, plan, implement e review) começa com o Supervisor solicitando ao Cursor que abra uma nova sessão, com contexto limpo, sem exceção. O handoff entre fases passa sempre pelo Supervisor, que dispara a sessão nova.

**Caminho rápido (Dev Expresso):** o humano pede o ajuste, o Dev Expresso passa ao Supervisor, o Cursor executa em branch `fix/YYYYMMDD-nome-da-task` e sobe no dev, e o retorno volta ao humano pelo mesmo caminho. Branch e PR só com a aprovação do humano.

## 5. Comunicação bidirecional com o Cursor

**Ida.** Só o Supervisor lança agentes no Cursor, pela API v1 de agentes. O título do agente segue `slug-fase-request_id`. Ao lançar, o Supervisor grava o **ID do agente** no livro-razão, junto do pedido.

**Volta.** O canal oficial é a notificação de conclusão, sem webhook. Quando o agente termina, o Cursor envia uma **notificação de conclusão do agente de nuvem**, que acorda o Supervisor. Exemplo real (teste `req-20261103-001`, trechos cortados com …):

```
Background task "Cloud agent bc-e3674a19-1e58-53b1-bbfa-eb88e2276620" (cursor-agent) finished:
The Cursor agent finished.
Agent: https://cursor.com/agents/bc-e3674a19-1e58-53b1-bbfa-eb88e2276620
No pull request link is available yet (the agent may have made no changes, or the PR is still being created).

Summary from the Cursor agent:
A branch `fix/20261103-versao-adega-v1-1` está no remoto, com o commit `44836f8`. …
<img src="/opt/cursor/artifacts/faixa-producao.png" alt="…" />
<img src="/opt/cursor/artifacts/faixa-vite-dev.png" alt="…" />
## Arquivos … ## Testes e build … ## Como ver …
```

| Vem na notificação | Não vem |
| --- | --- |
| Status de conclusão (`finished`) | slug |
| ID do agente (`bc-…`), no título e no link | `request_id` |
| Resumo com branch, commit, arquivos, testes e como ver | ID do modelo usado |
| Aviso de PR ainda não disponível | duração da execução |
| Duas imagens (prints do agente) | link de PR |

**Casamento por ID do agente.** O Supervisor extrai o ID do título ou do link e procura exatamente esse ID no livro-razão. Com um único pedido casado, marca como entregue e repassa ao Dev Expresso (ou ao bot da fase) o resumo e ao humano o aviso com as imagens. Nunca casa por nome, slug, branch ou repositório.

**Repasse ao Dev Expresso (decisão de 2026-11-05).** Quando a solicitação parte do Dev Expresso e o Supervisor a recebe, ele replica a mensagem da mesma forma que recebeu, sem reinterpretar, sem formato fixo de `BRANCH:` e `COMMIT:` e sem anexar imagens por conta própria.

**Rede de segurança.** A varredura de status consulta cada pedido aberto por ID e run, cobrindo erro, timeout e notificação perdida. Uma checagem diária de órfãos, que lista os agentes do Cursor e compara com o livro-razão (com filtro de janela de 24 horas, repositórios da fábrica e prefixo do título para não dar falso alarme, e só avisando), está **recomendada e ainda não implementada**.

Detalhes completos: `markdown/automacoes/comunicacao-bidirecional-cursor.md`.

## 6. Pedido órfão (seção 9 da `supervisor-regras`)

Se uma notificação de conclusão chegar e o ID do agente não casar com nenhum registro do livro-razão (ID não encontrado, ou gravação que falhou no lançamento):

1. O Supervisor avisa o humano, no chat dele, com os dados da notificação: branch, commit, arquivos e o link com o ID do agente.
2. Nunca entrega a ninguém sem casamento único.
3. O humano identifica o pedido manualmente, pela branch (padronizada com data por solicitação) ou indo ao repositório.

## 7. Seções por feature

**Seção** é o agrupamento de bots na barra lateral. Não é sessão de execução do Cursor.

### Criação

O Supervisor inicia o script local `criar-secao-feature.py`. O companion de UI cria a seção com o nome normalizado da feature a partir da Âncora, devolve somente a Âncora à seção Fábrica Agêntica e confere que a seção nova ficou vazia. O script **não cria bots**. Só depois do sucesso confirmado (exit 0, relatório `CONCLUIDO`, Âncora devolvida, seção vazia) o Supervisor pede ao Dr. Eggbot que crie o time nessa seção, resolvida por `ListSections` real. Contrato: `contratos/contrato-webhook-criacao.md`.

A rotina receptora **Fábrica — bloqueio criação** só registra o resultado (sucesso ou falha) e avisa o Elienaldo. Não cria bots, não executa scripts, não inicia o handoff ao Dr. Eggbot e não libera operação.

### Destruição

Só o Supervisor inicia o script local `destruir-secao-feature.py`, com autorização humana explícita de alvo e alcance e com capacidades reais validadas. O Dr. Eggbot **não recebe esse pedido**. Fim de feature, merge ou aviso de conclusão, por si só, não autorizam destruir. Contrato: `contratos/contrato-webhook-destruicao.md`.

A rotina receptora **Fábrica — bloqueio destruição** só registra bloqueio de preflight (exit 2) e avisa o Elienaldo. Não exclui, não libera flags e não corrige automaticamente.

**Estado honesto da destruição (conferido em 11/11/2026):**

- `C:\Users\Elienaldo\GrokBot-Scripts\destruir-secao-config.json` está com `validated=false`, evidências nulas e flags de inventário não liberadas. O modo real permanece bloqueado.
- Em 08/11/2026, 22:46 (America/Bahia), o ensaio autorizado na seção `Teste` (Cirurgião) passou no inventário (`inventory_verified`) e falhou em seguida, no modo `OpenDialogCancel`, ao identificar o menu contextual (`ProgrammaticName` ausente no objeto). `destructive_submit_count=0`. Nenhum bot ou seção foi excluído.
- Candidatos de correção permaneceram apenas em staging (`auditoria-preparacao-t1089\candidato` e `backup-staging-t1091`). Não foram promovidos à configuração de produção.

## 8. Padrões adotados

- **Branch por solicitação, com data:** toda nova solicitação ganha branch nova, nunca a `main` e nunca a de outra solicitação. Padrão `feat/YYYYMMDD-nome-da-task` ou `fix/YYYYMMDD-nome-da-task`. O slug sai das palavras principais do pedido, com até uns três termos. Exemplo real: `fix/20261103-versao-adega-v1-1`.
- **Modelo padrão: o modelo atual (default) do Cursor**, sem nomear versão, porque a fábrica usa o Cursor como background de desenvolvimento e planejamento. Se o humano pedir outro modelo no início, ele segue em todas as fases; a troca no meio do fluxo não é automática.
- **Modelo com créditos esgotados:** se o humano pedir um modelo cujos créditos já acabaram, o Supervisor (e o Dev Expresso) avisa que ele não pode ser usado, sugere o modelo padrão atual (default) do Cursor e não lança com o modelo esgotado.
- **Status por ID:** sempre por ID de agente e por run, nunca consulta global.
- **Sem agente real em teste sem o ok do humano; sem expor a chave `CURSOR_API_KEY`; sem relançar em laço após erro.**
- **Visualização de UI:** o prompt de lançamento exige URL pública (túnel) quando o agente sobe servidor de desenvolvimento; sem ela, o Supervisor avisa que só há prints. A VM do agente não tem tempo de vida garantido: ver na hora da entrega.
- **Criação de bots sempre pelo modelo:** cada bot nasce da ficha em `markdown/modelos/`. Sem ficha, o Dr. Eggbot não cria. Quem pede a criação cola o texto do modelo na solicitação; o Dr. Eggbot não acessa diretamente a pasta local do projeto.
- **Backup antes de alterar:** todo documento alterado ganha cópia de backup; só valem os arquivos sem sufixo `-anterior`.
- **Limites de rodadas, contados separadamente:** research 2, implement 3, the-judge 3 por execução, Cirurgião 3.
- **Validação humana do protótipo antes do plan** (seção 22 de `supervisor-regras`): consenso da equipe não substitui o OK explícito do Elienaldo. Sem esse OK, o handoff para o plan fica bloqueado.
- **Regra de ouro de atualização:** toda atualização de regra de um bot exige, no mesmo trabalho, atualizar a biblioteca carregável (skill ou perfil) e a documentação do projeto, com backup e hash antes/depois. Documento e biblioteca não podem ficar em versões diferentes.

## 9. Como utilizar

**Ajuste pequeno (caminho rápido):**
1. Fale com o Dev Expresso, no app do Grok Bot, descrevendo o ajuste e o projeto.
2. Ele confirma o que entendeu, aplica o freio de escopo e repassa ao Supervisor.
3. O Supervisor lança o agente do Cursor em branch nova e acompanha.
4. Você recebe, pelo Dev Expresso, o resumo, os prints, os arquivos, os testes e os links. Dê o retorno em bate-volta até aprovar.
5. Só então autorize PR e merge.

**Feature (fluxo padrão):**
1. Entregue a demanda ao Supervisor (ticket, texto ou outra fonte). Informe o projeto, se ele perguntar.
2. Opcional: diga o nível na solicitação ("aceite como média") e o modelo desejado.
3. O Supervisor cria a seção pelo script local e, só depois do sucesso, pede a equipe ao Dr. Eggbot.
4. Acompanhe pelo Supervisor, que é o único que fala com você. Decida quando ele escalar (impasse, blocker, risco, validação do protótipo).
5. No merge, receba o aviso. Se quiser destruir a seção, a autorização precisa ser explícita quanto ao alvo e ao alcance. Hoje o script de destruição continua bloqueado: o Supervisor informa o bloqueio e não inicia exclusão real enquanto `validated=false`.

**Quando algo não casar:** vale a seção 6. Procure o pedido pela branch ou pelo repositório.

## 10. Estrutura de pastas

```
Fabrica-Agentica/
├── README.md
├── skills/
│   └── supervisor-regras.md          # regras gerais do Supervisor (seções 1 a 27)
├── contratos/
│   ├── contrato-webhook-criacao.md
│   ├── contrato-webhook-destruicao.md
│   └── execucao-verificacao/
│       └── fluxo-contrato-verificacao.md
├── excalidraw/
│   ├── research/  plan/  implement/  review/
│   ├── contrato/
│   ├── acompanhamento/
│   ├── gerenciamento-secoes-por-feature.excalidraw
│   └── LINKS.md                      # links excalidraw.com (gerados em 05/11/2026)
├── markdown/
│   ├── comunicacao.md
│   ├── prompt-research-enxuto.md
│   ├── decisoes/                     # registros de decisão (ex.: seções por feature)
│   ├── fases/                        # registro em texto de cada fase
│   ├── modelos/                      # ficha de cada bot e visão geral da fábrica
│   ├── regras/                       # convenções e regra de ouro
│   └── automacoes/                   # comunicação com o Cursor (receptora de webhook inativa)
├── backup/                           # cópias de segurança (detalhe omitido aqui)
└── registros-auditoria/              # trilhas de auditoria (detalhe omitido aqui)
```

Há também o diretório `.git`. Arquivos com sufixo `-anterior` (em qualquer pasta) são backups e não valem.

Os scripts `criar-secao-feature.py` e `destruir-secao-feature.py`, a configuração e os logs de ensaio ficam em `C:\Users\Elienaldo\GrokBot-Scripts`, não neste repositório.

## 11. O que cada documento cobre

| Documento | O que cobre |
| --- | --- |
| `markdown/modelos/fábrica-agêntica.md` | Visão geral: as 8 fases, regras gerais (sessão por feature, consenso, classificação, limites, regras para o humano, roteamento de skills), política de modelos, as skills (incluindo o Verifier na TLC Spec Lean) e o ciclo de vida da squad. |
| `markdown/modelos/<bot>.md` | Ficha de cada bot: nome, job único, anti-jobs, voz, quando acorda, como concreto e ciclo de vida. Bots: Supervisor, Dev Expresso, Dr. Eggbot, Principal Product Engineer, Product Engineer, Design de front-end, Advogado do diabo, Orquestrador de pipeline, Curador amigável, Cirurgião e Âncora. |
| `markdown/modelos/dashboard-benchmarks.html` | Painel de benchmarks de modelos. |
| `markdown/comunicacao.md` | Grupos de chat (permanente, direção, research, review), pares de comunicação, roteamento, nome dos canais e histórico. |
| `markdown/fases/fase-*.md` | Registro em texto dos diagramas de cada fase (research, plan, implement, review). As versões `fase-research-v1.md` e `fase-plan-v1.md` são histórico. |
| `markdown/regras/convencoes-fabrica.md` | Padrão de branch, roteamento por grupo, campos que a notificação de conclusão não traz, criação de bots pelo modelo, modelo padrão e regra de ouro. |
| `markdown/decisoes/gerenciamento-secoes-por-feature-2026-11-06.md` | Registro da ligação de 06/11/2026 sobre seções por feature. É documento histórico; trechos ainda descrevem os scripts como futuros (ver pendências). |
| `markdown/automacoes/comunicacao-bidirecional-cursor.md` | Fluxo real da ida e da volta com o Cursor: passo a passo, payload, o que vem e o que não vem, casamento, pedido órfão e varredura. |
| `markdown/automacoes/automacao-receptora-cursor.md` | Desenho da receptora única de webhooks do Cursor. Inativa, sem uso previsto por enquanto (futuro/opcional); fora do fluxo. |
| `contratos/contrato-webhook-criacao.md` | Contrato operacional da criação de seção e do aviso à receptora **Fábrica — bloqueio criação**. |
| `contratos/contrato-webhook-destruicao.md` | Contrato operacional da destruição de seção e do aviso de bloqueio à receptora **Fábrica — bloqueio destruição**. |
| `contratos/execucao-verificacao/fluxo-contrato-verificacao.md` | Contrato de execução e verificação: as 8 fases, quem faz, o artefato de cada uma e os limites de rodadas. |
| `skills/supervisor-regras.md` | Regras do Supervisor (seções 1 a 27), inclusive workspace, canal com o humano, sessão nova por fase, validação do protótipo, criação e destruição local de seção, e research. |
| `excalidraw/**/*.excalidraw` | Diagramas de cada fase, do contrato, do acompanhamento da comunicação e do gerenciamento de seções. |

## 12. Pendências e pontos em aberto

- Script de destruição bloqueado (`validated=false`). O teste real de 08/11/2026 parou no menu contextual; nada foi excluído; candidatos de correção continuam só em staging. Flags não devem ser liberadas sem testes autorizados.
- Testes reais do fluxo padrão com o projeto Adega foram previstos a partir de 2026-11-05; este repositório não registra conclusão ponta a ponta desse fluxo.
- Checagem diária de órfãos (recomendada, não implementada).
- Confirmar no teste real como o push cria o PR (configuração do Cursor ou do GitHub) e se a notificação traz o link do PR (cadeia da implement à review, `comunicacao-bidirecional-cursor.md`, seção 8).
- Pontos em aberto de `comunicacao.md`: confirmar o prefixo `feat`, o dono da linha de veredito no ADR, e a abertura/fechamento dos grupos além do research.
- **Divergência:** a visão atual em `fábrica-agêntica.md` classifica com 5 perguntas; o contrato de verificação (e o README anterior) ainda fala em 3 perguntas.
- **Divergência:** `markdown/decisoes/gerenciamento-secoes-por-feature-2026-11-06.md` ainda trata criação e destruição como scripts futuros; os contratos e os scripts locais já existem, com a ordem operacional atual (criar seção vazia via Âncora, depois pedir os bots ao Dr. Eggbot).
- **Divergência:** o contrato de verificação resume a saída do research como design doc; a visão geral distingue PRD (média) e design doc (alta).
- O contrato de verificação ainda aponta um link Excalidraw antigo e marca o link novo como pendente; `excalidraw/LINKS.md` já lista os links de 05/11/2026.
