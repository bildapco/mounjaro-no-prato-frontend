# Contexto Completo — Frontend Mounjaro no Prato

## Por que esse repositório existe

O projeto começou em um único repositório (`Protocolo-Mounjaro-no-Prato`) que tinha frontend e backend juntos, hospedado no Railway. Em julho de 2026 decidimos separar em dois repositórios independentes para:

- Poder trocar a plataforma de hospedagem de cada parte sem afetar a outra
- Facilitar o trabalho paralelo entre os desenvolvedores
- Reduzir custo (Railway estava sendo pago; Netlify + Render são gratuitos)

O frontend foi para este repositório: `bildapco/mounjaro-no-prato-frontend`  
O backend foi para: `bildapco/mounjaro-no-prato-backend`

**Este é o único repositório de frontend ativo. O código do frontend no repositório antigo está abandonado.**

---

## Onde está publicado

- **Plataforma**: Netlify (plano gratuito)
- **Domínio**: https://protocolomp.com.br
- **Deploy**: automático a cada push na branch `main`
- **Build**: sem build command — os arquivos são servidos direto (site estático)

---

## Estrutura de arquivos

```
mounjaro-no-prato-frontend/
├── index_html.html                  ← Página de vendas / landing page
├── mini-app-mounjaro-no-prato.html  ← App de membros (área logada)
├── sw.js                            ← Service worker (cache offline / PWA)
├── manifest.json                    ← PWA manifest
├── _redirects                       ← Roteamento do Netlify
├── netlify.toml                     ← Configuração do Netlify (publish = ".")
├── .gitignore
└── Images/                          ← Imagens do site
```

---

## Roteamento

O arquivo `_redirects` controla como o Netlify serve as páginas:

```
/       /index_html.html                    200
/home   /mini-app-mounjaro-no-prato.html   200
/*      /index_html.html                    302
```

| URL | Arquivo servido |
|-----|----------------|
| `protocolomp.com.br/` | `index_html.html` (landing page) |
| `protocolomp.com.br/home` | `mini-app-mounjaro-no-prato.html` (app de membros) |
| qualquer outra URL | redireciona para a landing page |

---

## Como fazer deploy

```bash
git add .
git commit -m "descrição da alteração"
git push origin main
# O Netlify detecta o push e publica em segundos
```

> ⚠️ **IMPORTANTE**: O Netlify do plano gratuito só aceita deploys de commits feitos pela conta vinculada (`bildapco` / Igor). Se você fizer push com sua conta pessoal (`filipecerezofcs`), o deploy vai falhar com erro "unrecognized Git contributor". O código vai para o GitHub normalmente, mas o Netlify não publica. A solução é sempre combinar com o Igor para ele fazer o push final, ou ele puxa suas alterações e faz o push.

---

## netlify.toml

Arquivo obrigatório. Sem ele o Netlify ignora a configuração de diretório de publicação e não serve os arquivos corretamente.

```toml
[build]
  publish = "."
```

O `.` indica que a raiz do repositório é o diretório publicado — sem build step.

---

## Service Worker (sw.js)

O service worker faz cache dos arquivos do app para funcionar offline (PWA). Ele tem uma regra importante:

```javascript
if (e.request.url.includes('onrender.com')) return; // não cacheia API
```

Isso garante que as chamadas para o backend no Render nunca sejam cacheadas — sempre vão para a rede.

---

## Conexão com o Backend

A URL da API fica em `mini-app-mounjaro-no-prato.html`, linha ~4591:

```javascript
var AUTH_API = 'https://mounjaro-no-prato-backend.onrender.com';
```

Se o backend mudar de endereço, é só atualizar essa variável.

---

## Histórico de migrações

| Data | O que aconteceu |
|------|----------------|
| Jun/2026 | Projeto criado no Railway (frontend + backend juntos) |
| Jul/2026 | Frontend separado para `bildapco/mounjaro-no-prato-frontend` |
| Jul/2026 | Netlify conectado ao novo repo, domínio `protocolomp.com.br` migrado |
| Jul/2026 | Backend migrado para Render + Neon PostgreSQL |

---

## Repositório do Backend

Para entender a API, endpoints, variáveis de ambiente e fluxo da Hotmart, consulte:

```
https://github.com/bildapco/mounjaro-no-prato-backend
```

Arquivo de contexto: `docs/contexto-projeto.md`
