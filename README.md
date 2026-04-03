# daviaviss — blog posts

> Posts do meu blog pessoal em Markdown, consumidos pelo [portfólio](https://github.com/daviaviss/personal-portfolio) via GitHub API.

![posts](https://img.shields.io/badge/posts-4-blue?style=flat-square)
![idiomas](https://img.shields.io/badge/idiomas-PT%20%2B%20EN-green?style=flat-square)

---

## Como funciona

O portfólio busca os arquivos diretamente deste repositório via GitHub API. Cada post é um `.md` com frontmatter YAML. Posts bilíngues têm uma versão `.en.md` — se não houver, o portfólio usa o `.md` como fallback.

---

## Estrutura de um post

```md
---
title: "Título Do Post Com Capitalização Em Cada Palavra"
description: "Descrição curta exibida na listagem"
date: "YYYY-MM-DD"
tags: ["tag1", "tag2"]
---

Conteúdo em Markdown...
```

---

## Convenção de nomes

| Arquivo | Idioma |
|---|---|
| `slug.md` | Português (padrão) |
| `slug.en.md` | Inglês |

Slugs em kebab-case, sem acentos. Exemplo: `como-usar-hooks-react`.

---

## Posts

| Post | PT | EN |
|---|---|---|
| Como Estruturar Navegação no React Native | [PT](como-estruturar-navegacao-react-native.md) | [EN](como-estruturar-navegacao-react-native.en.md) |
| Server Components no Next.js | [PT](server-components-nextjs.md) | [EN](server-components-nextjs.en.md) |
| Autenticação com NextAuth.js | [PT](autenticacao-nextauth.md) | [EN](autenticacao-nextauth.en.md) |
| Tailwind CSS na Prática | [PT](tailwind-css-na-pratica.md) | [EN](tailwind-css-na-pratica.en.md) |

---

## Publicar um novo post

```bash
# 1. Criar os arquivos
touch slug-do-post.md slug-do-post.en.md

# 2. Commitar e fazer push
git add .
git commit -m "Add slug-do-post article (PT + EN)"
git push origin main
```

O portfólio atualiza automaticamente a cada hora.
