---
title: "Meu Primeiro Post"
description: "Este é o primeiro post do blog usando Nuxt Content"
date: "2024-01-15"
author: "Quila"
tags: ["nuxt", "vue", "blog"]
---

# Meu Primeiro Post

Bem-vindo ao meu blog! Este é o primeiro post criado usando **Nuxt Content**.

## O que é Nuxt Content?

Nuxt Content é um módulo oficial do Nuxt que permite criar um CMS baseado em arquivos markdown, YAML, CSV ou JSON.

### Características principais:

- ✅ Suporte a Markdown
- ✅ Front-matter com metadados
- ✅ Busca e filtros
- ✅ API REST automática
- ✅ Tipagem TypeScript

## Como usar

```typescript
// Buscar todos os posts
const posts = await queryContent('posts').find()

// Buscar um post específico
const post = await queryContent('posts').where({ slug: 'primeiro-post' }).findOne()
```

Espero que você goste deste post!
