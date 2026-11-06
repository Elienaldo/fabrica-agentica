# Comunicação bidirecional GrokBot e Cursor (fase research)

**Data:** 2026-11-02
**Status:** ida e volta definidas; receptor de teste criado; falta a chave da API e o teste ponta a ponta.

## Ida: GrokBot lança o agente no Cursor

- `POST https://api.cursor.com/v1/agents`, autenticação Bearer ou Basic com chave de conta de serviço (escopo `agent`).
- Um despachante único monta a chamada. O Principal dispara a fase research (`mode: plan`); o Orquestrador dispara plan, implement e review.
- `name` do agente carrega o rótulo de roteamento: `slug|fase|request_id`. O Cursor não tem campo de skill nem de metadados, então a skill vai no texto do prompt.
- `repos[0].startingRef` aponta para a branch de feature, nunca para a main: `feat/YYYYMMDD-nome-da-task`.
- A chave nunca fica em chat, arquivo ou prompt; fica como segredo do GrokBot.

## Volta: Cursor avisa o GrokBot

- Status por consulta (`GET /v1/agents/{id}/runs/{runId}`) ou por stream (SSE). Webhook na API v1 ainda "em breve".
- Na versão antiga da API (v0) há webhook por agente, só para `FINISHED` e `ERROR`, assinado com HMAC. Payload: event, timestamp, id, status, source, target (branchName, prUrl), summary.
- O webhook não traz "tipo" nem slug. Os dois vêm do despachante GrokBot, ao lançar o agente (slug no `name`; tipo derivado do `status`).
- Timeout e intervenção não existem como evento do webhook.

## Receptor

- Definitivo: automação única no bot supervisor (ver `automacao-receptora-cursor.md`). Não criado ainda.
- Teste: rotina "Receptor de teste: webhook do Cursor (fábrica agêntica)", no Mestre de obras, com gatilho de webhook. Só mostra o payload ao Elienaldo no chat; não envia a grupos, não lança agentes, não altera arquivos.
- A URL do webhook e a chave de envio ficam no painel da rotina; o Elienaldo as copia de lá. Nunca em arquivo.

## Pendente

1. Chave de conta de serviço do Cursor guardada como segredo do GrokBot.
2. Colar a URL do webhook no ponto de saída do Cursor e disparar um agente de teste.
3. Receptor definitivo no supervisor, depois do teste.
