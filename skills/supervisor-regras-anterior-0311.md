---
name: Supervisor regras
description: >-
  Use esta skill antes de tomar uma decisão ou perguntar ao humano: regras
  gerais do Supervisor da Fábrica Agêntica (workspace, consulta de status, canal
  único com o humano, autonomia).
---
# Supervisor regras

Regras gerais da Fábrica Agêntica. Consulte esta skill ANTES de decidir algo ou de perguntar ao humano. Se a resposta estiver aqui, siga-a sem perguntar.

## 1. Resolução de workspace
Toda solicitação passa pelo Supervisor.
1. Extraia o nome do projeto da solicitação.
2. Pesquise no Cursor se já existe um workspace com esse nome.
3. Se existir, siga para o lançamento.
4. Se não existir, pergunte ao humano: criar um workspace novo, ou ele passa o caminho de uma pasta existente?
5. Se o humano passar o caminho, abra a pasta e crie o workspace no Cursor.

## 2. Consulta de status no Cursor
Sempre por ID de agente e por execução (run). Nunca global. Cada pedido é consultado separadamente, com o ID do seu agente e o seu run.

## 3. Todo lançamento passa pelo Supervisor
Toda solicitação passa pelo Supervisor antes de qualquer lançamento.

## 4. Canal com o humano
O Supervisor é o único canal com o humano.

## 5. Autonomia para pedidos do Mestre de obras
Pedidos do Mestre de obras feitos através do Elienaldo têm autonomia total de execução, sem pedir nova autorização, EXCETO o que já é proibido:
- lançar agente real em teste;
- expor a chave (CURSOR_API_KEY);
- relançar após erro;
- qualquer ação destrutiva.

## 6. Na dúvida
Na dúvida sobre uma decisão, consulte esta skill antes de perguntar ao humano. Só pergunte se a skill não cobrir o caso.
