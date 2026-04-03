# daviaviss — blog-posts

Posts do meu blog pessoal em Markdown, consumidos pelo [portfólio](https://github.com/daviaviss/personal-portfolio) via GitHub API.

## Como funciona

O portfólio busca os posts diretamente deste repositório usando a API do GitHub. Cada post é um arquivo `.md` com frontmatter YAML. Posts com versão em inglês usam o sufixo `.en.md`.

## Estrutura de um post

```md
---
title: Título do post
description: Descrição curta exibida na listagem
date: 2024-01-01
tags: [tag1, tag2]
---

Conteúdo em Markdown...
```

## Convenção de nomes

| Arquivo | Idioma |
|---|---|
| `slug.md` | Português (padrão) |
| `slug.en.md` | Inglês |

Se não houver versão `.en.md`, o portfólio exibe o arquivo `.md` como fallback para leitores em inglês.

## Posts

| Post | PT | EN |
|---|---|---|
| Como estruturar navegação no React Native | [ver](como-estruturar-navegacao-react-native.md) | [ver](como-estruturar-navegacao-react-native.en.md) |
| Server Components no Next.js | [ver](server-components-nextjs.md) | [ver](server-components-nextjs.en.md) |
| Autenticação com NextAuth.js | [ver](autenticacao-nextauth.md) | [ver](autenticacao-nextauth.en.md) |
| Tailwind CSS na prática | [ver](tailwind-css-na-pratica.md) | [ver](tailwind-css-na-pratica.en.md) |