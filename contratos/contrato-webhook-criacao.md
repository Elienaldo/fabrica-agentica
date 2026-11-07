# Contrato operacional — criação de seção e aviso ao Supervisor

## Responsáveis e sequência

1. Supervisor recebe a demanda autorizada, escolhe o nome da feature e executa no PC Windows:

```powershell
py -3 "C:\Users\Elienaldo\GrokBot-Scripts\criar-secao-feature.py" "cadastro-fornecedor"
```

2. O argumento posicional é `nome_feature`. O script aplica NFC, remove espaços externos e troca `/` e `\` por hífen: `cadastro/fornecedor` vira `cadastro-fornecedor`. Nome vazio, controles/formatos invisíveis/surrogates e nomes reservados são rejeitados. A seção de destino deve ser inexistente; Grok Bot aberto em desktop interativo desbloqueado e Âncora única na seção Fábrica Agêntica.
3. O companion UIA move Âncora para uma nova seção, renomeia para o nome normalizado, devolve somente Âncora à Fábrica Agêntica e verifica que a seção nova permanece vazia. Nenhum bot é criado pelo script. Não é uma sessão Cursor.
4. Ao terminar a operação, o wrapper registra resultado e faz automaticamente uma tentativa de aviso final ao Supervisor, tanto em sucesso quanto em falha capturável com nome disponível. Não exige disparo manual separado. O evento informa seção, exit e erro. Não há retry da criação nem do aviso.
5. Só após sucesso confirmado da criação — exit0 e relatório CONCLUIDO, com Âncora devolvida e seção vazia — o Supervisor solicita ao Dr Eggbot criar o time diretamente nessa seção. Dr Eggbot consulta `ListSections` real, resolve o nome exato sem ambiguidade e obtém o ID antes de `CreateAgent`. O wrapper retorna `section_id=null`: nunca fornece nem inventa ID backend.

Esta sequência é o procedimento operacional. O script implementa criação/UI e aviso; não implementa execução agendada pelo Supervisor nem chamada automática ao Dr Eggbot. A receptora somente registra resultado e notifica Elienaldo; não cria bots, executa scripts, desenvolve, exclui ou libera ações.

## Saída e falhas

Stdout JSON e `logs/criar-secao-<uuid>-final.json` contêm execution_id estável, section_name normalizado, section_id, status da criação, exit_code e notification. O relatório original do companion continua em `logs/criar-secao-<uuid>.json`. O mesmo execution_id acompanha o evento final.

Exit0 significa criação concluída; 3 falha operacional; 2 argumentos inválidos; 124 timeout; 130 interrupção capturada. Código não zero do companion nunca é apresentado como sucesso, mesmo se seu relatório disser CONCLUIDO. Em falha/posição incerta, conferir o estado antes de qualquer nova execução; a seção pode já ter sido parcialmente criada. Não repetir para resolver falha de comunicação.

Falha do envio NÃO transforma criação bem-sucedida em criação falha: o exit da criação é preservado e notification/needs_human_notification expõem a lacuna. `--help` não executa operação nem envia aviso. Se não houver nome não vazio/seguro após erro de argumentos, registra `not_sent_invalid_section`, sem fabricar seção ou enviar payload inválido. Encerramento forçado do processo não garante aviso.

## Canal exclusivo da criação

Supervisor, rotina `f-brica-bloqueio-cria-o`.

URL: https://api2.cursor.sh/automations/webhook/1fdfe002-fccf-52a1-a49a-1348065506e2

Configuração: `FABRICA_CRIACAO_WEBHOOK_URL` e `FABRICA_CRIACAO_WEBHOOK_KEY`. O helper usa ambiente do processo; variáveis ausentes podem ser obtidas somente dessas mesmas entradas Windows User. Nunca usa variáveis/chave da destruição. Sender key só por canal seguro; nunca no fonte, argumentos, chat ou documentos.

POST JSON, Content-Type application/json, Authorization Bearer sender key e X-Automation-Key com a mesma chave. Timeout8s, uma tentativa, sem retry/redirecionamento. Chave/URL e controles são removidos do erro enviado. Sem credencial configurada: not_configured e nenhum POST.

## Contrato recebido

- execution_id: string não vazia, única por execução;
- script: literal criar-secao-feature.py;
- secao: string não vazia, nome normalizado;
- status: sucesso ou falha;
- exit: inteiro, nunca string/booleano;
- erro: string; vazio em sucesso, não vazio em falha.

Sucesso exige status=sucesso, exit=0 e erro="". Falha exige status=falha, exit diferente de zero e erro não vazio. Campos inconsistentes/incompletos exigem intervenção, nunca são inferidos como sucesso. A receptora mantém deduplicação de melhor esforço por execution_id, sem garantia exactly-once, e histórico independente em `/workspace/fabrica-bloqueio-criacao/eventos.jsonl`.

HTTP200 é aceitação pelo endpoint, não prova de aviso humano/recebimento confirmado: delivery_confirmed permanece false no remetente. Timeout ou erro de rede pode deixar entrega incerta; o remetente não reenvia automaticamente. Supervisor correlaciona o execution_id antes de continuar o procedimento.
