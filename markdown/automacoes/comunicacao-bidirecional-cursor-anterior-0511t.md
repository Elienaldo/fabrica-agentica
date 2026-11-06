# Comunicação bidirecional GrokBot e Cursor

**Data:** 2026-11-04
**Status:** funcionando de ponta a ponta. Validado no teste real `req-20261103-001` (Dev Expresso, Supervisor, Cursor, Supervisor, Dev Expresso). Substitui a versão de 2026-11-02, que descrevia webhook e chave faltando.
**Canal oficial de retorno:** a notificação de conclusão do agente de nuvem. O webhook do Cursor **não é usado** na fábrica.

## 1. Resumo

- **Ida:** só o Supervisor lança agentes no Cursor, pela API v1 de agentes (ele usa a ferramenta de agente de nuvem do Cursor, que já tem a chave).
- **Volta:** o retorno chega ao Supervisor como **notificação de conclusão do agente de nuvem**, que acorda o próprio Supervisor. Esse é o canal oficial e o único caminho comprovado no teste real.
- **Casamento:** o Supervisor grava o ID do agente no livro-razão ao lançar e cruza esse ID quando a notificação chega.
- **Webhook:** não é usado e não é necessário. A API v1 não tem webhook. A rotina receptora de webhook do Supervisor e a receptora de teste do Mestre de obras ficam **inativas, sem uso previsto por enquanto** (futuro/opcional, nunca rodaram). Nada no fluxo depende delas.

## 2. Fluxo completo, passo a passo

1. O Dev Expresso (ou o Orquestrador, nas fases) pede ao Supervisor uma execução, com `request_id`, slug, fase e o texto do pedido.
2. O Supervisor resolve o workspace do projeto e confere as regras (skill `supervisor-regras`: modelo com créditos esgotados, política de VM etc.).
3. O Supervisor lança o agente no Cursor, informando repositório, título e prompt. O título segue o padrão `slug-fase-request_id`. O prompt manda trabalhar numa branch padronizada com data, por solicitação (exemplos reais: `fix/20261103-versao-adega-v1-1`, `feat/YYYYMMDD-nome-da-task`), nunca na main, e não abrir PR nem fazer merge sem ordem do Elienaldo (regra do caminho rápido; no fluxo padrão, o push ao final da implement segue a seção 8).
4. O Cursor devolve o **ID do agente** (`bc-…`). O Supervisor grava no livro-razão, junto do pedido: `request_id`, slug, fase e **ID do agente**. Sem essa gravação não há casamento.
5. O agente trabalha na VM do Cursor. A varredura de status (seção 6) acompanha pelo ID de cada pedido aberto.
6. Quando o agente termina, o Cursor envia a **notificação de conclusão**, que acorda o Supervisor (seção 3).
7. O Supervisor lê o ID no título e no link, cruza com o livro-razão (seção 4) e, se houver casamento único, marca o pedido como entregue (commit, PR vazio) e repassa ao Dev Expresso a mensagem como recebeu (resumo com branch, commit, arquivos, testes e como ver), sem reinterpretar, sem formato fixo e sem anexar imagens por conta própria.
8. O Supervisor também avisa o Elienaldo no chat dele, com as imagens do agente anexadas.
9. O Dev Expresso entrega ao Elienaldo e aguarda a ordem dele (por exemplo, abrir PR).

**Decisão (2026-11-05):** Quando a solicitação parte do Dev Expresso e o Supervisor a recebe, ele replica a mensagem da mesma forma que recebeu, sem reinterpretar, sem formato fixo de `BRANCH:` e `COMMIT:` e sem anexar imagens por conta própria.

Se o ID não casar, vale a seção 5.

## 3. Notificação de conclusão: o que vem e o que não vem

Exemplo literal (teste `req-20261103-001`, trechos longos cortados com …):

```
Background task "Cloud agent bc-e3674a19-1e58-53b1-bbfa-eb88e2276620" (cursor-agent) finished:
The Cursor agent finished.
Agent: https://cursor.com/agents/bc-e3674a19-1e58-53b1-bbfa-eb88e2276620
No pull request link is available yet (the agent may have made no changes, or the PR is still being created).

Summary from the Cursor agent:
A branch `fix/20261103-versao-adega-v1-1` está no remoto, com o commit `44836f8`. Não abri pull request e não fiz merge.
…
<img src="/opt/cursor/artifacts/faixa-producao.png" alt="…" />
<img src="/opt/cursor/artifacts/faixa-vite-dev.png" alt="…" />
## Arquivos …
## Testes e build …
## Como ver …
```

**O que vem:**

| Campo | Onde aparece |
| --- | --- |
| Status | "finished" na primeira linha. Só foi visto o caso de conclusão; como chegam erro e timeout ainda não foi observado, por isso a varredura cobre esses casos |
| ID do agente (`bc-…`) | no título da notificação e no link `Agent: https://cursor.com/agents/bc-…` |
| Resumo escrito pelo agente | branch, commit, arquivos alterados, testes e build, como ver a aplicação |
| Aviso de PR | linha informando que ainda não há link de pull request |
| Imagens | duas imagens (prints do agente), copiadas para a pasta do Supervisor; o caminho `/opt/cursor/artifacts/…` do texto só existe na VM do agente |

**O que NÃO vem:**

- slug e `request_id` (só existem no título que o Supervisor deu ao agente e no livro-razão; não confiar no nome, porque o slug também usa traço);
- ID do modelo usado;
- duração da execução;
- link de PR (o agente não abre PR sem ordem; o aviso de "PR ainda não disponível" é padrão);
- campos estruturados: o resumo é texto livre escrito pelo agente. Por isso o prompt de lançamento deve pedir a branch e o commit em linhas fixas.

## 4. Como o Supervisor casa o pedido

1. No lançamento, grava o **ID do agente** no livro-razão, junto do `request_id`.
2. Quando a notificação chega, extrai o ID (título ou link) e procura **exatamente** esse ID no livro-razão.
3. Se achar um único pedido, entrega a esse pedido e só a ele. Nunca casa por nome, slug, branch ou repositório.

## 5. Pedido órfão (notificação sem casamento)

Regra oficial, seção 9 da skill `supervisor-regras`:

- Se a notificação chegar e o ID do agente não casar com nenhum registro do livro-razão (ID não encontrado ou gravação que falhou no lançamento), o Supervisor **avisa o Elienaldo**, no chat dele, com os dados da notificação: branch, commit, arquivos e o link com o ID do agente.
- **Nunca entrega a ninguém sem casamento único.**
- O **Elienaldo identifica o pedido manualmente**, pela branch (padronizada com data por solicitação) ou indo ao repositório.
- O Supervisor não tenta adivinhar o pedido por branch ou repositório e não relança nada.

## 6. Varredura de status: rede de segurança

- **Hoje:** a rotina "Varredura de status dos agentes do Cursor" roda a cada cerca de 10 minutos. Sem pedido aberto, sai em silêncio sem consultar o Cursor. Com pedido aberto, consulta cada um pelo ID do agente e pelo run (`GET /v1/agents/{id}/runs/{runId}`) e cobre o caso de a notificação não chegar (erro, timeout, notificação perdida).
- **Recomendado, ainda não implementado:** uma checagem de órfãos, menos frequente (diária, por exemplo), que lista os agentes do Cursor (`GET /v1/agents`) e compara os IDs com o livro-razão. A lista traz agentes de fora da fábrica, então precisa de filtros para não dar falso alarme: janela de tempo pela data de criação (últimas 24 horas), repositórios da fábrica e, para os agentes novos, prefixo do título `slug-fase-request_id`. Só avisa o Elienaldo; não entrega, não relança e não altera o livro-razão.
- Também recomendado: alerta separado quando existir pedido aberto no livro-razão sem ID de agente (lançamento que falhou antes de gravar).
- Qualquer mudança na rotina só com o ok do Elienaldo.

## 7. Limites conhecidos

- A notificação não traz o ID do modelo nem a duração: a plataforma escolhe o modelo, e a conferência real é pelo painel de uso.
- O dev server e o preview ficam na VM do agente, em endereços locais; sem túnel público o Elienaldo só vê os prints.
- Pendências de decisão do Elienaldo: modelo explícito no lançamento; checagem diária de órfãos.
- Decididos em 2026-11-05 (não são mais pendência): replicar a mensagem ao Dev Expresso como recebida, sem formato fixo de `BRANCH:` e `COMMIT:` e sem anexar imagens por conta própria.

## 8. Do fim da implementação ao início da review

Decidido em 2026-11-05.
1. O modelo do Cursor que executa a implementação, ao final de uma implementação sem erros (verificação da skill ok), executa o push no GitHub. Quem faz o push é o modelo do Cursor, nunca o Supervisor. Se a verificação falhar, não faz push.
2. O push cria o PR automaticamente.
3. O modelo avisa o Supervisor, pela notificação de conclusão do agente, que o push foi feito e o PR foi criado.
4. O Supervisor, sabendo que o PR foi criado, avisa o Orquestrador de pipeline que a fase de implementação foi finalizada e que o PR existe, com o número do PR.
5. O Orquestrador pede ao Supervisor que crie uma nova sessão no Cursor e inicie a fase de review, pedindo o comando da `the-judge` com o número do PR.

**Push e PR não conflitam com a regra de não abrir PR sem ordem.** O push vai sempre para a branch da task (`feat/` ou `fix/YYYYMMDD-nome-da-task`) e nunca para a `main`. A regra do prompt de lançamento, de não abrir PR nem fazer merge sem ordem do Elienaldo, vale para o caminho rápido (Dev Expresso) e para qualquer ação fora desta cadeia. No fluxo padrão, a orientação de push vem do Supervisor no pedido da fase de implementação (seção 12 de `supervisor-regras`), e o merge continua sendo disparado só pelo Orquestrador, depois da review.

**A confirmar no teste real:** o mecanismo que cria o PR a partir do push (configuração do Cursor ou do GitHub) e se a notificação traz o link do PR.

## 9. Depois do merge: ambiente dev

Decidido em 2026-11-05. É uma fase nova, depois do merge.
1. O Orquestrador faz o merge no GitHub (automático, com a review limpa).
2. Ao final do merge, o Orquestrador solicita ao Supervisor que envie uma solicitação ao modelo do Cursor que fez a sessão de review, sem abrir sessão nova, para subir o ambiente dev.
3. O modelo sobe o dev, gera o link do ambiente e devolve ao Supervisor, pela notificação de conclusão do agente (o Supervisor casa pelo ID do agente da review, que está no livro-razão).
4. O Supervisor avisa o Elienaldo, no chat dele, que o dev está pronto, com o link. A comunicação com o humano é sempre do Supervisor, nunca do Orquestrador.
5. O Elienaldo testa no dev e, se precisar de ajustes, solicita ao modelo pelo Supervisor.
6. O build de prod (produção) só acontece depois que o Elienaldo validar.

O Orquestrador não precisa ser avisado de que o dev foi gerado. Ainda não existe um ambiente dev estruturado (é uma feature do backlog); o dev atual é provisório.

**A confirmar no teste real:** se a ferramenta de agente de nuvem do Supervisor permite enviar nova solicitação ao agente da review depois de ele terminar, e se o link do dev fica acessível (vale a regra de URL pública por túnel da ficha do Supervisor; sem URL pública, o Supervisor avisa que só há prints).

## 10. Insumos das fases: tudo passa pelo Supervisor

Decidido em 2026-11-05. Toda comunicação entre atores passa pelo Supervisor.
1. O Principal Product Engineer entrega os insumos (design doc, PRD ou solicitação direta) ao Supervisor. Não existe caminho direto do Principal ao Orquestrador.
2. O Supervisor repassa os insumos ao Orquestrador e determina se é plan ou implement conforme o insumo (planejamento ou implementação). No plan, o Orquestrador acorda quando o Supervisor inicia a fase.
3. O Orquestrador, que já tem as regras dele, ao receber os insumos solicita de volta ao Supervisor que faça a solicitação ao Cursor.
4. O Supervisor faz a solicitação ao Cursor (seção 2). O retorno vem pela notificação de conclusão do agente, que o Supervisor casa pelo ID.

Caminho: Principal → Supervisor → Orquestrador → Supervisor → Cursor.
