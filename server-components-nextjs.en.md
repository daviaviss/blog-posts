---
title: "Understanding React Server Components in Next.js 14"
description: "How Server Components change the way you think about performance and architecture in Next.js apps — and when to use them in practice."
date: "2026-04-02"
tags: ["next.js", "react", "server components", "performance"]
---

One of the most impactful changes in Next.js 14 with the App Router is the adoption of **React Server Components** by default. If you're coming from the Pages Router, this shift can feel strange at first — but once you understand the logic, it's hard to go back.

## What Are Server Components?

Server Components are React components that run **exclusively on the server**. They are never sent to the client, which means:

- No JavaScript from them in the client bundle
- Direct access to databases, file systems, and environment variables
- Data fetching without `useEffect` or external libraries

In the App Router, **every component is a Server Component by default**. To use hooks, events, or state, you need to declare `"use client"` at the top of the file.

## Server vs Client: When to Use Each

| Situation | Server Component | Client Component |
|---|---|---|
| Fetch data from API or database | ✅ | ❌ |
| Access environment variables | ✅ | ❌ |
| Use `useState`, `useEffect` | ❌ | ✅ |
| User interactions (click, input) | ❌ | ✅ |
| Use React contexts | ❌ | ✅ |

## Fetching Data Directly

In the App Router, you can fetch data directly in the component, without needing `getServerSideProps` or `getStaticProps`:

```tsx
// app/posts/page.tsx — Server Component by default
async function getPosts() {
  const res = await fetch("https://api.example.com/posts", {
    next: { revalidate: 60 }, // revalidates every 60 seconds
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

No hooks, no state, no manual loading. The component is async and data arrives ready.

## Composing Server and Client Components

The most important rule: **Client Components cannot directly import Server Components**, but Server Components can import Client Components normally.

The correct pattern for passing server data to interactive components is via props:

```tsx
// app/dashboard/page.tsx — Server Component
import UserCard from "@/components/UserCard"; // Client Component

async function getUser(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}`);
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
        {expanded ? "Show less" : "Show more"}
      </button>
      {expanded && <p>{user.bio}</p>}
    </div>
  );
}
```

## Cache and Revalidation

Next.js 14 has a granular cache system. You control the behavior of each `fetch` individually:

```tsx
// No cache — always fresh data
fetch(url, { cache: "no-store" });

// Permanent cache (default)
fetch(url);

// Revalidate every N seconds (ISR)
fetch(url, { next: { revalidate: 60 } });

// Revalidate by tag (on-demand)
fetch(url, { next: { tags: ["posts"] } });
```

To manually invalidate the cache by tag, use `revalidateTag` in a Server Action:

```ts
"use server";

import { revalidateTag } from "next/cache";

export async function refreshPosts() {
  revalidateTag("posts");
}
```

## Practical Tips

- **Keep as much as possible as Server Components** — only add `"use client"` when you truly need interactivity
- **Push `"use client"` to the leaves of the tree** — the lower in the hierarchy, the smaller the client bundle
- **Use Suspense for loading states** — wrap async Server Components in `<Suspense>` for progressive streaming
- **Prefer native `fetch`** — Next.js extends `fetch` with automatic caching, no need for Axios or React Query for basic data

Server Components represent a real paradigm shift. With them, you write less code, send less JavaScript to the client, and have a cleaner separation between what is server logic and what is user interaction.