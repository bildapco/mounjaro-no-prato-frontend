# Teste de Desativação de Usuário via Hotmart

## Contexto

API .NET 10 hospedada no Railway:
```
https://protocolo-mounjaro-no-prato-production.up.railway.app
```

O webhook já está configurado no backend. Ao receber um evento de reembolso/cancelamento da Hotmart, ele automaticamente:
- Seta `isActive: false` no usuário
- Revoga todos os refresh tokens (desloga de todos os dispositivos)

---

## Passo 1 — Verificar o webhook na Hotmart

No painel da Hotmart → **Ferramentas** → **Webhooks**, confirmar:

- **URL:** `https://protocolo-mounjaro-no-prato-production.up.railway.app/api/hotmart/webhook`
- **Eventos selecionados:**
  - `PURCHASE_APPROVED`
  - `PURCHASE_COMPLETE`
  - `PURCHASE_REFUNDED`
  - `PURCHASE_CHARGEBACK`
  - `PURCHASE_CANCELLED`

---

## Passo 2 — Verificar a lista de usuários

```
GET https://protocolo-mounjaro-no-prato-production.up.railway.app/admin/users?recoveryCode=JV1A9N8TS2KTP6U38WA6WD4U
```

Se o usuário aparecer com `"isActive": false`, a desativação funcionou.

---

## Passo 3 — Verificar logs no Railway

No painel do Railway → serviço da API → aba **Logs**.

Quando a Hotmart disparar o webhook, aparece uma linha com o request. Se não aparecer nada, o webhook não está chegando.

---

## Possíveis Problemas e Soluções

### Webhook retorna 401 (Unauthorized)
O `HOTMART__WEBHOOKSECRET` no Railway não bate com o secret configurado na Hotmart.

**Solução:** copiar o secret exato da Hotmart e atualizar a variável `HOTMART__WEBHOOKSECRET` no Railway.

---

### Webhook retorna 200 mas usuário continua ativo
O email do comprador no payload da Hotmart não coincide com o email cadastrado no banco.

**Solução:** fazer o GET de usuários e comparar o email exato.

---

### Webhook não chega (sem logs no Railway)
O evento não está selecionado na Hotmart, ou a URL está errada.

**Solução:** editar o webhook na Hotmart e confirmar URL e eventos.

---

## Desativação e Reativação Manual

Para testes ou casos manuais (sem precisar da Hotmart):

**Desativar:**
```bash
curl -X POST https://protocolo-mounjaro-no-prato-production.up.railway.app/admin/deactivate-user \
  -H "Content-Type: application/json" \
  -d '{"recoveryCode":"JV1A9N8TS2KTP6U38WA6WD4U","email":"email-do-usuario@gmail.com"}'
```

**Reativar:**
```bash
curl -X POST https://protocolo-mounjaro-no-prato-production.up.railway.app/admin/activate-user \
  -H "Content-Type: application/json" \
  -d '{"recoveryCode":"JV1A9N8TS2KTP6U38WA6WD4U","email":"email-do-usuario@gmail.com"}'
```
