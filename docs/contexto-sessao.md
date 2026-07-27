# Contexto da Sessão — Decisões e Problemas Resolvidos

Este arquivo registra o histórico de decisões técnicas, problemas encontrados e soluções aplicadas durante o desenvolvimento e migração do projeto.

---

## Migração do Projeto (Jul/2026)

### Por que migramos

O projeto estava em um único repositório (`Protocolo-Mounjaro-no-Prato`) com frontend e backend juntos, hospedado no Railway (pago). Migramos para:

- **Frontend** → `bildapco/mounjaro-no-prato-frontend` no Netlify (gratuito, sem sleep)
- **Backend** → `bildapco/mounjaro-no-prato-backend` no Render (gratuito, dorme após 15min)
- **Banco de dados** → Railway SQLite → Neon PostgreSQL (gratuito, sem expiração)

---

## Problema: Netlify bloqueando commits do Cerezo

**Problema**: O Netlify no plano gratuito só aceita deploys de commits feitos pela conta vinculada (`bildapco` / Igor). Commits do Cerezo (`filipecerezofcs`) chegam ao GitHub normalmente mas o Netlify recusa o deploy com erro "unrecognized Git contributor".

**Status**: Sem solução gratuita. O fluxo adotado é o Cerezo fazer commit e o Igor fazer o push final, ou o Igor puxar as alterações do Cerezo e fazer o push.

---

## Problema: Render dormindo após 15 minutos

**Problema**: O Render no plano gratuito suspende o serviço após 15 minutos de inatividade. A primeira requisição após o sleep demora ~30-50 segundos.

**Solução**: Configurado o **UptimeRobot** (gratuito) para pingar o endpoint `/health` do backend a cada 5 minutos, mantendo o serviço acordado.

- URL monitorada: `https://mounjaro-no-prato-backend.onrender.com/health`
- O plano gratuito do Render tem 750h/mês por serviço (equivale a rodar 24/7 o mês inteiro), então manter acordado não gera custo extra.

**Observação**: Quando há um novo deploy no Render, o serviço fica ~1 minuto retornando 502 enquanto o novo container sobe. O UptimeRobot detecta e envia email — é comportamento esperado, não é problema.

---

## Problema: PWA não atualizava após deploy

**Problema**: Usuários que instalaram o app na tela inicial do celular não recebiam atualizações automaticamente. O service worker servia os arquivos do cache local mesmo após um novo deploy.

**Causa**: O `sw.js` tinha o nome do cache fixo (`mnp-v2`). Como o nome não mudava, o browser não detectava que havia uma nova versão.

**Solução**: Criado um hook de pre-commit em `.git/hooks/pre-commit` que atualiza automaticamente o nome do cache com timestamp a cada commit:

```sh
#!/bin/sh
DATE=$(date +%Y%m%d%H%M)
sed -i "s/const CACHE = 'mnp-[^']*';/const CACHE = 'mnp-$DATE';/" sw.js
git add sw.js
```

Agora a cada commit o `sw.js` recebe uma versão como `mnp-202607271044`. O browser detecta o novo service worker, apaga o cache antigo e baixa os arquivos frescos.

**Comportamento após deploy**: O usuário precisa fechar o app completamente e reabrir para a atualização ser aplicada.

---

## Problema: Deploy 502 durante restart do Render

**Problema**: A cada novo push no backend, o Render derruba o container atual e sobe o novo. Durante esse processo (~1 minuto) o serviço retorna 502.

**Status**: Sem solução no plano gratuito. É comportamento esperado e documentado.

---

## Fluxo de trabalho atual

### Para atualizar o frontend
1. Cerezo faz as alterações e commita localmente
2. Cerezo faz push para o GitHub (`git push`)
3. Igor puxa as alterações (`git pull`) e faz o push final — ou Igor faz o push diretamente
4. Netlify detecta o push do Igor e publica em segundos
5. O hook de pre-commit atualiza o cache do service worker automaticamente

### Para atualizar o backend
1. Qualquer um dos dois faz push para `bildapco/mounjaro-no-prato-backend`
2. Render detecta e faz o deploy automaticamente (~2 minutos)
3. O serviço fica ~1 minuto com 502 durante a troca de container

---

## Endpoints úteis de administração

```
GET  /admin/users?recoveryCode=JV1A9N8TS2KTP6U38WA6WD4U
POST /admin/activate-user    { recoveryCode, email }
POST /admin/deactivate-user  { recoveryCode, email }
POST /admin/setup-user       { recoveryCode, email, newPassword }
GET  /health
```

---

## Monitoramento

- **UptimeRobot**: monitora o `/health` do backend a cada 5 minutos e envia email para `bildapco` em caso de queda
- **Admin users**: `GET /admin/users` mostra todos os usuários e seus status em tempo real

---

## Clientes reais no banco

Os usuários com IDs 1 a 8 são contas de teste criadas durante o desenvolvimento. O primeiro cliente real foi:

- **ID 9** — MARIELA RODRIGUES COSTA (`marii_rcosta@hotmail.com`) — cadastrada via webhook da Hotmart em Jul/2026
