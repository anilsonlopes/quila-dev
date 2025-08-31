<template>
  <div class="min-h-screen bg-gray-50 py-8">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <!-- Header -->
      <div class="text-center mb-12">
        <h1 class="text-4xl font-bold text-gray-900 mb-4">
          Blog Posts
        </h1>
        <p class="text-xl text-gray-600 max-w-2xl mx-auto">
          Explore todos os posts do nosso blog. Encontre conteúdo sobre Nuxt, Vue, desenvolvimento web e muito mais.
        </p>
      </div>

      <!-- Filtros -->
      <div class="mb-8 flex flex-wrap gap-4 justify-center">
        <button
          v-for="tag in allTags"
          :key="tag"
          @click="toggleTag(tag)"
          :class="[
            'px-4 py-2 rounded-full text-sm font-medium transition-colors',
            selectedTags.includes(tag)
              ? 'bg-blue-600 text-white'
              : 'bg-white text-gray-700 border border-gray-300 hover:bg-gray-50'
          ]"
        >
          {{ tag }}
        </button>
      </div>

      <!-- Loading state -->
      <div v-if="pending" class="text-center py-12">
        <div class="inline-block animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
        <p class="mt-4 text-gray-600">Carregando posts...</p>
      </div>

      <!-- Posts Grid -->
      <div v-else-if="filteredPosts.length > 0" class="grid gap-8 md:grid-cols-2 lg:grid-cols-3">
        <article
          v-for="post in filteredPosts"
          :key="post._path"
          class="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow duration-300"
        >
          <!-- Post Image (placeholder) -->
          <div class="h-48 bg-gradient-to-br from-blue-400 to-purple-500 flex items-center justify-center">
            <div class="text-white text-6xl font-bold opacity-20">
              {{ (post.title || 'Post').charAt(0) }}
            </div>
          </div>

          <!-- Post Content -->
          <div class="p-6">
            <!-- Tags -->
            <div class="flex flex-wrap gap-2 mb-3">
              <span
                v-for="tag in post.tags"
                :key="tag"
                class="px-2 py-1 bg-blue-100 text-blue-800 text-xs rounded-full font-medium"
              >
                {{ tag }}
              </span>
            </div>

            <!-- Title -->
            <h2 class="text-xl font-bold text-gray-900 mb-2 line-clamp-2">
              {{ post.title || 'Sem título' }}
            </h2>

            <!-- Description -->
            <p class="text-gray-600 mb-4 line-clamp-3">
              {{ post.description }}
            </p>

            <!-- Meta -->
            <div class="flex items-center justify-between text-sm text-gray-500 mb-4">
              <span class="flex items-center">
                <svg class="w-4 h-4 mr-1" fill="currentColor" viewBox="0 0 20 20">
                  <path fill-rule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clip-rule="evenodd" />
                </svg>
                {{ post.author }}
              </span>
              <span class="flex items-center">
                <svg class="w-4 h-4 mr-1" fill="currentColor" viewBox="0 0 20 20">
                  <path fill-rule="evenodd" d="M6 2a1 1 0 00-1 1v1H4a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V6a2 2 0 00-2-2h-1V3a1 1 0 10-2 0v1H7V3a1 1 0 00-1-1zm0 5a1 1 0 000 2h8a1 1 0 100-2H6z" clip-rule="evenodd" />
                </svg>
                {{ formatDate(post.date) }}
              </span>
            </div>

            <!-- Read More Button -->
            <NuxtLink
              :to="`/posts/${post._path?.split('/').pop() || 'post'}`"
              class="inline-flex items-center px-4 py-2 bg-blue-600 text-white text-sm font-medium rounded-md hover:bg-blue-700 transition-colors duration-200"
            >
              Ler mais
              <svg class="w-4 h-4 ml-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
              </svg>
            </NuxtLink>
          </div>
        </article>
      </div>

      <!-- Empty state -->
      <div v-else class="text-center py-12">
        <div class="text-gray-400 text-6xl mb-4">📝</div>
        <h3 class="text-xl font-medium text-gray-900 mb-2">Nenhum post encontrado</h3>
        <p class="text-gray-600">
          {{ selectedTags.length > 0 
            ? 'Tente ajustar os filtros de tags.' 
            : 'Ainda não temos posts publicados.' 
          }}
        </p>
      </div>

      <!-- Stats -->
      <div v-if="posts && posts.length > 0" class="mt-12 text-center text-sm text-gray-500">
        Mostrando {{ filteredPosts.length }} de {{ posts.length }} posts
        <span v-if="selectedTags.length > 0">
          filtrados por {{ selectedTags.join(', ') }}
        </span>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
// Declaração para queryContent (Nuxt Content)
declare const queryContent: any

// Tipagem para os posts
interface Post {
  _path?: string
  title?: string
  description?: string
  date?: string
  author?: string
  tags?: string[]
}

// Buscar todos os posts
const { data: posts, pending } = await useAsyncData<Post[]>('posts', () => 
  queryContent('posts')
    .sort({ date: -1 }) // Ordenar por data (mais recente primeiro)
    .find()
)

// Estado para filtros de tags
const selectedTags = ref<string[]>([])

// Computed para obter todas as tags únicas
const allTags = computed(() => {
  if (!posts.value) return []
  
  const tags = new Set<string>()
  posts.value.forEach((post: Post) => {
    if (post.tags && Array.isArray(post.tags)) {
      post.tags.forEach((tag: string) => tags.add(tag))
    }
  })
  
  return Array.from(tags).sort()
})

// Computed para filtrar posts baseado nas tags selecionadas
const filteredPosts = computed(() => {
  if (!posts.value) return []
  
  if (selectedTags.value.length === 0) {
    return posts.value
  }
  
  return posts.value.filter((post: Post) => {
    if (!post.tags || !Array.isArray(post.tags)) return false
    return selectedTags.value.some(tag => post.tags!.includes(tag))
  })
})

// Função para alternar seleção de tags
const toggleTag = (tag: string) => {
  const index = selectedTags.value.indexOf(tag)
  if (index > -1) {
    selectedTags.value.splice(index, 1)
  } else {
    selectedTags.value.push(tag)
  }
}

// Função para formatar data
const formatDate = (dateString?: string) => {
  if (!dateString) return ''
  
  const date = new Date(dateString)
  return date.toLocaleDateString('pt-BR', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

// Meta tags para SEO
useHead({
  title: 'Blog Posts - Quila Dev',
  meta: [
    { name: 'description', content: 'Explore todos os posts do nosso blog sobre desenvolvimento web, Nuxt, Vue e muito mais.' },
    { property: 'og:title', content: 'Blog Posts - Quila Dev' },
    { property: 'og:description', content: 'Explore todos os posts do nosso blog sobre desenvolvimento web, Nuxt, Vue e muito mais.' },
    { property: 'og:type', content: 'website' }
  ]
})
</script>

<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.line-clamp-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>