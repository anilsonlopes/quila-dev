---
title: "Segundo Post - Dicas de Vue 3"
description: "Aprenda algumas dicas úteis para trabalhar com Vue 3 e Composition API"
date: "2024-01-16"
author: "Quila"
tags: ["vue", "composition-api", "dicas"]
---

# Dicas de Vue 3

Vue 3 trouxe muitas melhorias e a Composition API é uma das mais importantes.

## Composition API vs Options API

A Composition API oferece uma forma mais flexível de organizar a lógica do componente:

```vue
<script setup>
import { ref, computed, onMounted } from 'vue'

const count = ref(0)
const doubleCount = computed(() => count.value * 2)

const increment = () => {
  count.value++
}

onMounted(() => {
  console.log('Componente montado!')
})
</script>
```

## Vantagens da Composition API

1. **Reutilização de lógica**: Composables permitem compartilhar lógica entre componentes
2. **Melhor inferência de tipos**: TypeScript funciona melhor
3. **Organização por funcionalidade**: Agrupe lógica relacionada
4. **Tree-shaking**: Apenas o que você usa é incluído no bundle

## Exemplo de Composable

```typescript
// composables/useCounter.ts
export const useCounter = (initial = 0) => {
  const count = ref(initial)
  
  const increment = () => count.value++
  const decrement = () => count.value--
  const reset = () => count.value = initial
  
  return {
    count: readonly(count),
    increment,
    decrement,
    reset
  }
}
```

A Composition API é o futuro do Vue!
