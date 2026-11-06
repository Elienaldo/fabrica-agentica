# Convenções da fábrica agêntica

**Data:** 2026-11-02

## Branches

Toda nova solicitação cria uma branch nova. Nunca se reutiliza branch de outra solicitação.

**Padrão de nome:** `feat/YYYYMMDD-nome-da-task`

- `feat` é a abreviação de feature.
- `YYYYMMDD` é ano, mês e dia.
- `nome-da-task` é o slug gerado automaticamente pelo supervisor a partir das palavras principais da solicitação, sem artigos e preposições, com até uns três termos e hífen entre eles. O slug é o mesmo nome da task.

**Exemplo:** `feat/20261102-botao-login`

**Por quê:** isola uma solicitação da outra e deixa o merge limpo.

## Roteamento

O grupo de chat amarra o projeto. Se o pedido for de outro projeto, o supervisor pergunta antes de rotear.

## Campos que o webhook do Cursor não traz

O webhook do Cursor não traz o campo tipo nem o slug de origem. Os dois vêm do despachante GrokBot, ao lançar o agente. Quem preenche esses campos é o despachante, não o agente que roda.
