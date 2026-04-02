---
title: "Authentication With NextAuth.js in the App Router"
description: "How to set up full authentication in Next.js 14 using NextAuth.js — with social providers, sessions, and route protection."
date: "2026-04-02"
tags: ["next.js", "nextauth", "authentication", "security"]
---

Authentication is one of the most sensitive parts of any application. Getting it wrong can expose user data; getting it right can be tedious. **NextAuth.js** solves this elegantly, integrating natively with the Next.js App Router.

## What Is NextAuth.js?

NextAuth.js (also known as **Auth.js**) is an open-source library for authentication in Next.js applications. It supports dozens of providers — Google, GitHub, Discord, and more — as well as email/password authentication with custom credentials.

## Installation

```bash
npm install next-auth
```

## Basic Configuration

Create the configuration file at `auth.ts` in the project root:

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

Create the NextAuth API route at `app/api/auth/[...nextauth]/route.ts`:

```ts
// app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth";

export const { GET, POST } = handlers;
```

## Environment Variables

Add to `.env.local`:

```text
AUTH_SECRET=your_random_secret_key

GITHUB_CLIENT_ID=your_client_id
GITHUB_CLIENT_SECRET=your_client_secret

GOOGLE_CLIENT_ID=your_client_id
GOOGLE_CLIENT_SECRET=your_client_secret
```

To generate `AUTH_SECRET`:

```bash
npx auth secret
```

## Accessing the Session

In **Server Components**, use the `auth()` function directly:

```tsx
// app/dashboard/page.tsx
import { auth } from "@/auth";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const session = await auth();

  if (!session) redirect("/login");

  return (
    <div>
      <h1>Hello, {session.user?.name}!</h1>
      <p>Email: {session.user?.email}</p>
    </div>
  );
}
```

In **Client Components**, use the `useSession` hook:

```tsx
"use client";

import { useSession } from "next-auth/react";

export default function UserMenu() {
  const { data: session, status } = useSession();

  if (status === "loading") return <p>Loading...</p>;
  if (!session) return <p>Not authenticated</p>;

  return <p>Hello, {session.user?.name}!</p>;
}
```

## Protecting Routes With Middleware

The most efficient way to protect routes is via middleware — it runs at the edge before any rendering:

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

## Sign In and Sign Out Buttons

With Server Actions, the buttons are straightforward:

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
      <button type="submit">Sign in with GitHub</button>
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
      <button type="submit">Sign out</button>
    </form>
  );
}
```

## Final Tips

- **Never expose `AUTH_SECRET`** — always use environment variables
- **Use middleware for bulk protection** — it's more performant than checking on each page
- **Customize the session object via callbacks** — if you need extra data like `role` or `id`, add it in the `jwt` or `session` callback
- **Auth.js v5 is still in beta** — worth using in new projects, but check compatibility

NextAuth.js removes the complexity of implementing authentication from scratch. With just a few files you have social login, route protection, and sessions working in production.