---
title: "Entendendo React Server Components no Next.js 14"
description: "Como os Server Components mudam a forma de pensar em performance e arquitetura de aplicações Next.js — e quando usá-los na prática."
date: "2026-04-02"
tags: ["next.js", "react", "server components", "performance"]
---

Uma das mudanças mais impactantes do Next.js 14 com o App Router é a adoção dos **React Server Components** por padrão. Se você veio do Pages Router, essa mudança pode parecer estranha no início — mas depois que você entende a lógica, é difícil voltar.

## O Que São Server Components?

Server Components são componentes React que rodam **exclusivamente no servidor**. Eles nunca são enviados para o cliente, o que significa:

- Sem JavaScript deles no bundle do cliente
- Acesso direto a banco de dados, sistema de arquivos e variáveis de ambiente
- Busca de dados sem `useEffect` ou bibliotecas externas

No App Router, **todo componente é Server Component por padrão**. Para usar hooks, eventos ou estado, você precisa declarar `"use client"` no topo do arquivo.

## Server vs Client: Quando Usar Cada Um

| Situação | Server Component | Client Component |
|---|---|---|
| Buscar dados de API ou banco | ✅ | ❌ |
| Acessar variáveis de ambiente | ✅ | ❌ |
| Usar `useState`, `useEffect` | ❌ | ✅ |
| Interações do usuário (click, input) | ❌ | ✅ |
| Usar contextos React | ❌ | ✅ |

## Buscando Dados Diretamente

No App Router, você pode buscar dados diretamente no componente, sem precisar de `getServerSideProps` ou `getStaticProps`:

```tsx
// app/posts/page.tsx — Server Component por padrão
async function getPosts() {
  const res = await fetch("https://api.exemplo.com/posts", {
    next: { revalidate: 60 }, // revalida a cada 60 segundos
  });
  return res.json();
}

export default async function PostsPage() {
  const posts = await getPosts();

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

Sem hooks, sem estado, sem loading manual. O componente é assíncrono e os dados chegam prontos.

## Compondo Server e Client Components

A regra mais importante: **Client Components não podem importar Server Components diretamente**, mas Server Components podem importar Client Components normalmente.

O padrão correto para passar dados do servidor para componentes interativos é via props:

```tsx
// app/dashboard/page.tsx — Server Component
import UserCard from "@/components/UserCard"; // Client Component

async function getUser(id: string) {
  const res = await fetch(`https://api.exemplo.com/users/${id}`);
  return res.json();
}

export default async function DashboardPage() {
  const user = await getUser("123");

  return <UserCard user={user} />;
}
```

```tsx
// components/UserCard.tsx — Client Component
"use client";

import { useState } from "react";

export default function UserCard({ user }) {
  const [expanded, setExpanded] = useState(false);

  return (
    <div>
      <h2>{user.name}</h2>
      <button onClick={() => setExpanded(!expanded)}>
        {expanded ? "Ver menos" : "Ver mais"}
      </button>
      {expanded && <p>{user.bio}</p>}
    </div>
  );
}
```

## Cache e Revalidação

O Next.js 14 tem um sistema de cache granular. Você controla o comportamento de cada `fetch` individualmente:

```tsx
// Sem cache — dados sempre frescos
fetch(url, { cache: "no-store" });

// Cache permanente (padrão)
fetch(url);

// Revalida a cada N segundos (ISR)
fetch(url, { next: { revalidate: 60 } });

// Revalida por tag (on-demand)
fetch(url, { next: { tags: ["posts"] } });
```

Para invalidar o cache por tag manualmente, use `revalidateTag` em uma Server Action:

```ts
"use server";

import { revalidateTag } from "next/cache";

export async function refreshPosts() {
  revalidateTag("posts");
}
```

## Dicas Práticas

- **Mantenha o máximo possível como Server Components** — só adicione `"use client"` quando realmente precisar de interatividade
- **Mova o `"use client"` para as folhas da árvore** — quanto mais abaixo na hierarquia, menor o bundle do cliente
- **Use Suspense para loading states** — envolva Server Components assíncronos em `<Suspense>` para streaming progressivo
- **Prefira `fetch` nativo** — o Next.js estende o `fetch` com cache automático, sem precisar de Axios ou React Query para dados básicos

Server Components representam uma mudança de paradigma real. Com eles, você escreve menos código, envia menos JavaScript para o cliente e tem uma separação mais clara entre o que é lógica de servidor e o que é interação de usuário.