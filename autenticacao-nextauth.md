---
title: "Autenticação Com NextAuth.js no App Router"
description: "Como configurar autenticação completa no Next.js 14 usando NextAuth.js — com providers sociais, sessões e proteção de rotas."
date: "2026-04-02"
tags: ["next.js", "nextauth", "autenticação", "segurança"]
---

Autenticação é uma das partes mais sensíveis de qualquer aplicação. Fazer errado pode expor dados dos usuários; fazer bem pode ser trabalhoso. O **NextAuth.js** resolve isso de forma elegante, integrando nativamente com o App Router do Next.js.

## O Que É NextAuth.js?

NextAuth.js (agora também chamado de **Auth.js**) é uma biblioteca open-source para autenticação em aplicações Next.js. Ele suporta dezenas de providers — Google, GitHub, Discord, entre outros — além de autenticação por email/senha com credenciais customizadas.

## Instalação

```bash
npm install next-auth
```

## Configuração Básica

Crie o arquivo de configuração em `auth.ts` na raiz do projeto:

```ts
// auth.ts
import NextAuth from "next-auth";
import GitHub from "next-auth/providers/github";
import Google from "next-auth/providers/google";

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    }),
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
});
```

Crie a rota de API do NextAuth em `app/api/auth/[...nextauth]/route.ts`:

```ts
// app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth";

export const { GET, POST } = handlers;
```

## Variáveis de Ambiente

Adicione no `.env.local`:

```text
AUTH_SECRET=sua_chave_secreta_aleatoria

GITHUB_CLIENT_ID=seu_client_id
GITHUB_CLIENT_SECRET=seu_client_secret

GOOGLE_CLIENT_ID=seu_client_id
GOOGLE_CLIENT_SECRET=seu_client_secret
```

Para gerar o `AUTH_SECRET`:

```bash
npx auth secret
```

## Acessando a Sessão

Em **Server Components**, use a função `auth()` diretamente:

```tsx
// app/dashboard/page.tsx
import { auth } from "@/auth";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const session = await auth();

  if (!session) redirect("/login");

  return (
    <div>
      <h1>Olá, {session.user?.name}!</h1>
      <p>Email: {session.user?.email}</p>
    </div>
  );
}
```

Em **Client Components**, use o hook `useSession`:

```tsx
"use client";

import { useSession } from "next-auth/react";

export default function UserMenu() {
  const { data: session, status } = useSession();

  if (status === "loading") return <p>Carregando...</p>;
  if (!session) return <p>Não autenticado</p>;

  return <p>Olá, {session.user?.name}!</p>;
}
```

## Protegendo Rotas com Middleware

A forma mais eficiente de proteger rotas é via middleware — ele roda na borda antes de qualquer renderização:

```ts
// middleware.ts
import { auth } from "@/auth";

export default auth((req) => {
  const isLoggedIn = !!req.auth;
  const isProtected = req.nextUrl.pathname.startsWith("/dashboard");

  if (isProtected && !isLoggedIn) {
    return Response.redirect(new URL("/login", req.nextUrl));
  }
});

export const config = {
  matcher: ["/((?!api|_next/static|_next/image|favicon.ico).*)"],
};
```

## Botões de Login e Logout

Com Server Actions, os botões ficam simples:

```tsx
import { signIn, signOut } from "@/auth";

export function SignInButton() {
  return (
    <form
      action={async () => {
        "use server";
        await signIn("github");
      }}
    >
      <button type="submit">Entrar com GitHub</button>
    </form>
  );
}

export function SignOutButton() {
  return (
    <form
      action={async () => {
        "use server";
        await signOut();
      }}
    >
      <button type="submit">Sair</button>
    </form>
  );
}
```

## Dicas Finais

- **Nunca exponha o `AUTH_SECRET`** — use sempre variáveis de ambiente
- **Use middleware para proteção em massa** — é mais performático do que verificar em cada página
- **Customize o objeto de sessão via callbacks** — se precisar de dados extras como `role` ou `id`, adicione no callback `jwt` ou `session`
- **Auth.js v5 ainda está em beta** — em projetos novos vale a pena usar, mas verifique a compatibilidade

NextAuth.js remove a complexidade de implementar autenticação do zero. Com poucos arquivos você tem login social, proteção de rotas e sessões funcionando em produção.