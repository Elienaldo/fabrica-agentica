# Contrato operacional — destruição de seção e aviso de bloqueio

## Responsável, chamada e limites

Supervisor executa somente com autorização explícita para excluir os bots/seção da feature e após capacidades reais validadas:

```powershell
py -3 "C:\Users\Elienaldo\GrokBot-Scripts\destruir-secao-feature.py" "cadastro-fornecedor" --confirmar-exclusao "cadastro-fornecedor" --config "C:\Users\Elienaldo\GrokBot-Scripts\destruir-secao-config.json"
```

Nome deve estar NFC, sem espaços externos, e confirmação deve ser idêntica. Estado atual: modo real bloqueado por validated=false, evidências ausentes e inventário/capacidades não liberados. Configurar a notificação não libera exclusão. Não alterar booleans como substituto de evidência real.

## Sequência e proteção

Após preflight/capacidades e inventário completo sem ambiguidades, o adapter localiza a seção pelo nome exato, registra alvos e exclui seus bots antes de excluir a seção. Até duas passagens somente sobre remanescentes confirmados após reconciliação; nunca retry cego após submit/timeout incerto. Falha persistente interrompe antes de apagar seção. Se seção desaparecer, relê estado sem tentar outra seção.

Antes da seção, exige inventário vazio novamente e diálogo específico inequívoco. Seção Fábrica Agêntica/Uso Pessoal e bots permanentes Âncora/Supervisor/Dr Eggbot são protegidos. Âncora no alvo faz abortar antes de exclusão: deve ser devolvida à origem com autorização apropriada. Seção da sidebar não é sessão Cursor. Não confundir ocultação, movimento para Sem seção e exclusão de bot. RuntimeIds não são IDs backend estáveis.

Stdout JSON e log_dir/resultado.json informam status, stage, removed, remanescentes e needs_human_notification. Exit2 significa blocked_or_stopped/erro; exit0 somente sucesso reconciliado permitido pelo wrapper. Falha do processo filho não vira sucesso de JSON anterior. Nenhuma mensagem externa é enviada pelo próprio PowerShell; o hook Python descrito abaixo atende somente bloqueio inicial.

## Aviso automático existente

Somente stage=preflight e exit final2, fora do modo de inspeção, o wrapper tenta um aviso real ao Supervisor. Inclui qualquer motivo de bloqueio nessa etapa. Outros erros/stages permanecem saída/log com needs_human_notification, sem envio automático por esse hook.

Rotina exclusiva: f-brica-bloqueio-destrui-o.
URL: https://api2.cursor.sh/automations/webhook/59b27e72-59b5-56bf-b08d-d81486a626cf
Ambiente do processo: FABRICA_SUPERVISOR_WEBHOOK_URL e FABRICA_SUPERVISOR_WEBHOOK_KEY. Variáveis User persistentes precisam ser herdadas/carregadas por processos já abertos. Nunca usar chave/variáveis da criação.

POST JSON; Content-Type application/json; Authorization Bearer sender key e X-Automation-Key com a mesma chave. Timeout8s, uma tentativa por execução, sem retry/redirecionamento. Segredo nunca em fonte/argumentos/chat/log. Ausência de config: not_configured, sem envio; falha de transporte preserva exit2 e needs_human_notification. Sem outbox/reenvio automático.

## Payload

execution_id string única; script literal destruir-secao-feature.py; exit inteiro2; erro texto não vazio; status literal aguardando_humano. execution_id é o nome estável do log dessa execução. Repetir o CLI cria outra execução, não é retry do aviso anterior.

notification no JSON final de stdout distingue not_configured, invalid_configuration, http_accepted, http_rejected, send_failed e notifier_error. HTTP200 é somente aceitação; delivery_confirmed=false não comprova aviso humano ou recebimento confirmado. Supervisor deve correlacionar execução e avisar o humano; notificação não é autorização para nova tentativa de exclusão.
