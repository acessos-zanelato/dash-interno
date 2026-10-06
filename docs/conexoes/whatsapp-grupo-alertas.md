# Conexão WhatsApp — Grupo de Alertas

Registro da conexão usada para disparar alertas operacionais no WhatsApp.

## Identificação

| Campo           | Valor                                              |
| --------------- | -------------------------------------------------- |
| Nome do grupo   | 🚨 Alertas - Zanelato Marketing de Performance     |
| ID do grupo     | `120363351788933902-group`                         |
| Tipo            | Grupo do WhatsApp                                  |
| Finalidade      | Receber alertas automáticos do dash interno        |

## Formato do ID

O sufixo `-group` é o padrão da **Z-API** para identificar grupos. Outras
APIs usam formatos diferentes para o mesmo grupo:

| API / Provedor               | Formato                            |
| ---------------------------- | ---------------------------------- |
| Z-API                        | `120363351788933902-group`         |
| Evolution API / Baileys      | `120363351788933902@g.us`          |
| WhatsApp Web (JID nativo)    | `120363351788933902@g.us`          |

Use o formato correspondente ao provedor configurado. Enviar o ID no formato
errado retorna erro de destinatário inválido.

## Variável de ambiente

Não versionar o ID nem tokens no código. Configurar como variável/secret:

```bash
WHATSAPP_ALERTS_GROUP_ID=120363351788933902-group
```

Credenciais do provedor (ex.: `ZAPI_INSTANCE_ID`, `ZAPI_TOKEN`,
`ZAPI_CLIENT_TOKEN`) ficam apenas nos secrets do ambiente (Supabase Edge
Functions, Make ou servidor), nunca no repositório.

## Exemplo de envio (Z-API)

```bash
curl -X POST \
  "https://api.z-api.io/instances/$ZAPI_INSTANCE_ID/token/$ZAPI_TOKEN/send-text" \
  -H "Content-Type: application/json" \
  -H "Client-Token: $ZAPI_CLIENT_TOKEN" \
  -d '{
    "phone": "'"$WHATSAPP_ALERTS_GROUP_ID"'",
    "message": "🚨 Teste de alerta — dash interno"
  }'
```

Para grupos, o campo `phone` recebe o ID do grupo.

## Exemplo de envio (Evolution API)

```bash
curl -X POST "$EVOLUTION_URL/message/sendText/$EVOLUTION_INSTANCE" \
  -H "Content-Type: application/json" \
  -H "apikey: $EVOLUTION_API_KEY" \
  -d '{
    "number": "120363351788933902@g.us",
    "text": "🚨 Teste de alerta — dash interno"
  }'
```

## Padrão de mensagem de alerta

```
🚨 [SEVERIDADE] Cliente — Conta
Métrica: <métrica> = <valor> (limite: <limite>)
Período: <janela>
Ação sugerida: <ação>
```

Severidades: `CRÍTICO` (ação imediata), `ATENÇÃO` (revisar no dia),
`INFO` (registro).

## Checklist de validação

- [ ] Número conectado à instância é membro do grupo
- [ ] Número tem permissão para enviar mensagens (grupo não restrito a admins, ou número é admin)
- [ ] Instância do provedor com status conectado
- [ ] Mensagem de teste recebida no grupo
- [ ] Secrets configurados no ambiente de execução

## Pendências

- Confirmar qual provedor está em uso (Z-API, Evolution ou outro)
- Confirmar qual número/instância está conectado ao grupo
- Definir quais alertas do dash disparam para este grupo e seus limites
