# Automação receptora única de webhooks do Cursor

**Data:** 2026-11-02
**Bot:** GrokBot (supervisor)
**Função:** receptor único de toda a fábrica
**Status (2026-11-04): INATIVA, sem uso previsto por enquanto.** A rotina não foi apagada, mas nunca rodou e nada no fluxo depende dela. O canal oficial de retorno do Cursor é a notificação de conclusão do agente de nuvem (ver `comunicacao-bidirecional-cursor.md`). O webhook é futuro/opcional: só seria retomado se o Cursor passar a oferecer webhook e o Elienaldo decidir usá-lo.

## Descrição

Receptora única de webhooks do Cursor para toda a fábrica agêntica.

Existe uma só automação para a fábrica inteira. Todo webhook que o Cursor dispara chega aqui, e nenhuma fase, projeto ou grupo tem receptor próprio.

## Entrada

POST do Cursor com JSON que traz um campo "tipo", com um destes valores: conclusão, falha, timeout ou intervenção.

## O que faz, nesta ordem

Um. Valida o payload pela chave que está na URL. Se a chave não bater, descarta e não notifica ninguém.

Dois. Lê o slug da requisição no payload. É por ele que identifica o pedido e o grupo de destino.

Três. Roteia a notificação para o grupo certo e avisa o bot interno responsável pela fase daquele pedido, de acordo com o "tipo".

Quatro. Se o slug não existir ou não casar com nenhum pedido, avisa o supervisor, sem entregar a nenhum grupo.

## Segurança

A chave de validação fica só na URL, e nunca no código, no prompt ou em arquivos do repositório. A rotação da chave é manual, feita de vez em quando por um administrador, seguindo um checklist, porque a fábrica roda em ambiente corporativo controlado com acesso restrito. Ao rotacionar, a URL antiga deixa de valer assim que a nova for cadastrada no Cursor.

## Limites

Esta automação só recebe, valida e roteia. Ela não executa fases, não lança agentes e não decide o resultado de nada. A decisão continua com o bot da fase e com o supervisor.

## Notas

- Os passos um (descartar chave inválida) e quatro (slug desconhecido avisar o supervisor) foram acrescentados como reforço de segurança.
- O webhook do Cursor não traz o campo tipo nem o slug de origem. Os dois vêm do despachante GrokBot, ao lançar o agente. Quem preenche esses campos é o despachante, não o agente que roda.
- Em 2026-11-04 o webhook deixou de ser componente necessário da fábrica. Este desenho fica como referência futura e não deve ser tratado como parte do fluxo.
