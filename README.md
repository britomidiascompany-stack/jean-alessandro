# Jean Alessandro — Instalação de tracking

Guia de instalação do **Google Tag Manager** e do **subdomínio de server-side tagging (Stape.io)** no site publicado na Vercel.

> Contêiner de exemplo usado neste guia: `GTM-KFKMT6BZ`
> Domínio de exemplo usado neste guia: `fda-lotezero.vercel.app`

Troque os IDs e domínios de exemplo pelos valores reais do projeto quando for aplicar.

---

## Sumário

1. [Parte 1 — Instalar o snippet do GTM no código](#parte-1--instalar-o-snippet-do-gtm-no-código)
2. [Parte 2 — Apontar o subdomínio do servidor (Stape)](#parte-2--apontar-o-subdomínio-do-servidor-stape)
3. [Verificação final](#verificação-final)
4. [Problemas comuns](#problemas-comuns)

---

## Antes de começar

- **ID do contêiner GTM** (formato `GTM-XXXXXXX`), visível na tela "Instalar o Google Tag Manager" do seu contêiner.
- **Acesso ao repositório** do projeto com permissão para dar push (este repositório).
- **Saber qual tipo de projeto** está publicado — Next.js (App Router ou Pages Router) ou HTML/Vite/React puro. Se não souber, olhe o `package.json` na raiz do projeto.

---

## Parte 1 — Instalar o snippet do GTM no código

O GTM sempre exige dois blocos: um no `<head>` (o mais alto possível) e outro logo após a abertura do `<body>`. Onde colar cada um muda de acordo com o tipo de projeto.

### Next.js — App Router (`app/layout.tsx`)

Edite o **layout raiz** do projeto (não um layout de rota específica):

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="pt-BR">
      <head>
        {/* Google Tag Manager */}
        <script
          dangerouslySetInnerHTML={{
            __html: `(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
              new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
              j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
              'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
              })(window,document,'script','dataLayer','GTM-KFKMT6BZ');`,
          }}
        />
      </head>
      <body>
        {/* Google Tag Manager (noscript) */}
        <noscript>
          <iframe
            src="https://www.googletagmanager.com/ns.html?id=GTM-KFKMT6BZ"
            height={0} width={0}
            style={{ display: 'none', visibility: 'hidden' }}
          />
        </noscript>
        {children}
      </body>
    </html>
  );
}
```

> No App Router isso precisa estar no layout **raiz**. Se colar em um layout de subpasta, o GTM só carrega naquela seção do site.

### Next.js — Pages Router (`pages/_document.tsx`)

Crie o arquivo se ele ainda não existir:

```tsx
// pages/_document.tsx
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html lang="pt-BR">
      <Head>
        {/* Google Tag Manager */}
        <script
          dangerouslySetInnerHTML={{
            __html: `(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
              new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
              j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
              'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
              })(window,document,'script','dataLayer','GTM-KFKMT6BZ');`,
          }}
        />
      </Head>
      <body>
        {/* Google Tag Manager (noscript) */}
        <noscript>
          <iframe
            src="https://www.googletagmanager.com/ns.html?id=GTM-KFKMT6BZ"
            height={0} width={0}
            style={{ display: 'none', visibility: 'hidden' }}
          />
        </noscript>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}
```

> `_document.tsx` só existe uma vez por projeto e cobre todas as páginas automaticamente.

### HTML estático / Vite / React (`index.html`)

Cole exatamente como veio da tela do GTM, sem adaptar nada.

No `<head>`:

```html
<!-- Google Tag Manager -->
<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-KFKMT6BZ');</script>
<!-- End Google Tag Manager -->
```

Logo após a abertura do `<body>`:

```html
<!-- Google Tag Manager (noscript) -->
<noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-KFKMT6BZ"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
<!-- End Google Tag Manager (noscript) -->
```

### Suba a alteração

```bash
git add .
git commit -m "Instala Google Tag Manager"
git push
```

A Vercel republica sozinha assim que detecta o novo commit na branch de produção (normalmente `main`). Acompanhe em `vercel.com/dashboard` → o projeto → aba **Deployments**.

---

## Parte 2 — Apontar o subdomínio do servidor (Stape)

O container server-side precisa de um subdomínio próprio (ex.: `gtm.seudominio.com`) para rodar como first-party. Isso é feito **por DNS**, fora do código — não tem deploy nessa parte.

A tela de configuração do container server-side mostra uma tabela como a abaixo, com o subdomínio sugerido e o registro exato a criar (os valores mudam por projeto):

| Type  | Host                          | Value          |
|-------|-------------------------------|----------------|
| CNAME | `gtm.fda-lotezero.vercel.app` | `saf.stape.io` |

### 1. Adicione o registro CNAME no DNS do domínio

Onde exatamente depende de quem gerencia o DNS:

- **DNS gerenciado pela Vercel** — Vercel → o projeto → **Settings → Domains** → o domínio → gerenciar registros de DNS. Adicione **Host** e **Value** exatamente como na tabela.
- **Cloudflare** — aba **DNS → Records**, adicione o CNAME e deixe o **Proxy status desligado** (nuvem cinza — "DNS only").
- **Outro provedor** (Registro.br, GoDaddy, etc.) — mesma lógica: tipo CNAME, Host = subdomínio sugerido, Value = destino da tabela. Se houver campo TTL, deixe no padrão.

> ⚠️ **Ação necessária:** se o domínio estiver atrás do Cloudflare (ou similar), o **Proxy status precisa ficar OFF** nesse registro — com o proxy ligado, a verificação nunca conclui porque o CNAME some por trás do IP do Cloudflare.

### 2. Aguarde a verificação

A propagação do DNS mais a verificação automática podem levar **até 72 horas** — na prática costuma ser bem mais rápido. Nenhuma ação é necessária nesse meio-tempo.

---

## Verificação final

**A — No painel da Stape / GTM:** volte na tela onde pegou a tabela de DNS. O aviso de "Actions required" desaparece e o subdomínio passa a aparecer como verificado/conectado.

**B — Testando o CNAME direto:**

```bash
dig gtm.seudominio.com CNAME
```

(ou `nslookup gtm.seudominio.com` no Windows) e confirme que a resposta aponta para o valor da tabela.

**C — GTM Preview mode:** no Tag Manager, clique em **Visualizar**, cole a URL do site publicado e abra. Deve aparecer o painel **Connected** do Tag Assistant no rodapé da página.

**D — DevTools do navegador:** abra o site → `F12` → aba **Network** → filtre por `gtm`. Deve aparecer uma chamada para `googletagmanager.com/gtm.js` com status 200.

---

## Problemas comuns

<details>
<summary>O GTM aparece no Preview mas não no site publicado</summary>

O push provavelmente foi para uma branch diferente da que a Vercel usa como Production. Confira em `vercel.com/dashboard` → o projeto → **Settings → Git** qual branch está marcada como produção.
</details>

<details>
<summary>Não aparece nenhuma chamada para googletagmanager.com</summary>

- Dê um refresh forçado (`Cmd/Ctrl + Shift + R`) para ignorar cache do navegador.
- Confirme que o snippet foi salvo no arquivo certo e que o deploy mais recente terminou com status **Ready**.
- Bloqueadores de anúncio e extensões de privacidade podem esconder a chamada — teste em uma aba anônima sem extensões.
</details>

<details>
<summary>O GTM parece estar disparando duas vezes</summary>

Sinal de que o snippet foi colado em mais de um lugar — por exemplo no layout raiz **e** em um layout de rota específica. Deixe o bloco em um único arquivo, o mais alto possível na árvore do projeto.
</details>

<details>
<summary>Estou no App Router e não sei onde fica o &lt;head&gt;</summary>

O App Router não tem um arquivo `<head>` separado — o layout raiz (`app/layout.tsx`) já retorna a tag `<html>` inteira, e o `<head>` é escrito diretamente dentro dele, como no exemplo acima.
</details>

<details>
<summary>Já se passaram 72h e ainda aparece "Actions required" no Stape</summary>

- Confira se o **Host** foi digitado exatamente como na tabela (sem espaços, sem repetir o domínio duas vezes — alguns provedores já completam o domínio base sozinhos, então às vezes o campo Host é só a primeira parte, ex.: `gtm`).
- Se o domínio estiver no Cloudflare, confirme que o Proxy status está mesmo desligado (nuvem cinza, não laranja).
</details>

<details>
<summary>Não sei quem gerencia o DNS do domínio</summary>

Rode `whois seudominio.com` ou verifique em qual registrador o domínio foi comprado — os nameservers listados indicam onde os registros precisam ser adicionados (podem ser da Vercel, do Cloudflare, ou do próprio registrador).
</details>

<details>
<summary>O site usa apenas o domínio padrão *.vercel.app, não um domínio próprio</summary>

Vale confirmar com quem configurou o container server-side qual domínio de fato foi usado para o subdomínio sugerido — normalmente o server-side tagging é configurado sobre um domínio próprio do projeto, não sobre o domínio automático da Vercel.
</details>

---

*Brito Mídias · Central de Estratégia e Resultados · rafael@britomidias.com.br*
