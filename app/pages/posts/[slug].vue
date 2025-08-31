<template>
  <div class="min-h-screen bg-gray-50 py-8">
    <div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
      <!-- Loading state -->
      <div v-if="pending" class="text-center py-12">
        <div class="inline-block animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
        <p class="mt-4 text-gray-600">Carregando post...</p>
      </div>

      <!-- Post não encontrado -->
      <div v-else-if="!post" class="text-center py-12">
        <div class="text-gray-400 text-6xl mb-4">📝</div>
        <h1 class="text-2xl font-bold text-gray-900 mb-4">Post não encontrado</h1>
        <p class="text-gray-600 mb-8">O post que você está procurando não existe ou foi removido.</p>
        <NuxtLink
          to="/posts"
          class="inline-flex items-center px-6 py-3 bg-blue-600 text-white font-medium rounded-md hover:bg-blue-700 transition-colors duration-200"
        >
          ← Voltar para todos os posts
        </NuxtLink>
      </div>

      <!-- Post Content -->
      <article v-else class="bg-white rounded-lg shadow-lg overflow-hidden">
        <!-- Header -->
        <div class="p-8 border-b border-gray-200">
          <!-- Breadcrumb -->
          <nav class="mb-6">
            <NuxtLink
              to="/posts"
              class="text-blue-600 hover:text-blue-800 transition-colors duration-200"
            >
              ← Voltar para todos os posts
            </NuxtLink>
          </nav>

          <!-- Tags -->
          <div class="flex flex-wrap gap-2 mb-4">
            <span
              v-for="tag in post.tags"
              :key="tag"
              class="px-3 py-1 bg-blue-100 text-blue-800 text-sm rounded-full font-medium"
            >
              {{ tag }}
            </span>
          </div>

          <!-- Title -->
          <h1 class="text-4xl font-bold text-gray-900 mb-4">
            {{ post.title }}
          </h1>

          <!-- Meta -->
          <div class="flex items-center gap-6 text-sm text-gray-500">
            <span class="flex items-center">
              <svg class="w-4 h-4 mr-2" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clip-rule="evenodd" />
              </svg>
              {{ post.author }}
            </span>
            <span class="flex items-center">
              <svg class="w-4 h-4 mr-2" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M6 2a1 1 0 00-1 1v1H4a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V6a2 2 0 00-2-2h-1V3a1 1 0 10-2 0v1H7V3a1 1 0 00-1-1zm0 5a1 1 0 000 2h8a1 1 0 100-2H6z" clip-rule="evenodd" />
              </svg>
              {{ formatDate(post.date) }}
            </span>
          </div>

          <!-- Description -->
          <p v-if="post.description" class="mt-4 text-xl text-gray-600">
            {{ post.description }}
          </p>
        </div>

        <!-- Content -->
        <div class="p-8 prose prose-lg max-w-none">
          <ContentRenderer :value="post" />
        </div>

        <!-- Footer -->
        <div class="p-8 bg-gray-50 border-t border-gray-200">
          <div class="flex items-center justify-between">
            <div class="text-sm text-gray-500">
              <span>Postado em {{ formatDate(post.date) }}</span>
            </div>
            
            <!-- Share buttons -->
            <div class="flex gap-3">
              <button
                @click="sharePost"
                class="inline-flex items-center px-4 py-2 bg-blue-600 text-white text-sm font-medium rounded-md hover:bg-blue-700 transition-colors duration-200"
              >
                <svg class="w-4 h-4 mr-2" fill="currentColor" viewBox="0 0 20 20">
                  <path d="M15 8a3 3 0 10-2.977-2.63l-4.94 2.47a3 3 0 100 4.319l4.94 2.47a3 3 0 10.895-1.789l-4.94-2.47a3.027 3.027 0 000-.74l4.94-2.47C13.456 7.68 14.19 8 15 8z" />
                </svg>
                Compartilhar
              </button>
            </div>
          </div>
        </div>
      </article>

      <!-- Related Posts -->
      <div v-if="relatedPosts && relatedPosts.length > 0" class="mt-12">
        <h2 class="text-2xl font-bold text-gray-900 mb-6">Posts relacionados</h2>
        <div class="grid gap-6 md:grid-cols-2">
          <article
            v-for="relatedPost in relatedPosts"
            :key="relatedPost._path"
            class="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow duration-300"
          >
            <div class="p-6">
              <h3 class="text-lg font-bold text-gray-900 mb-2">
                {{ relatedPost.title }}
              </h3>
              <p class="text-gray-600 mb-4 line-clamp-2">
                {{ relatedPost.description }}
              </p>
              <NuxtLink
                :to="`/posts/${relatedPost._path?.split('/').pop() || 'post'}`"
                class="text-blue-600 hover:text-blue-800 font-medium transition-colors duration-200"
              >
                Ler mais →
              </NuxtLink>
            </div>
          </article>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
// Parâmetros da rota
const route = useRoute()
const slug = route.params.slug as string

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

// Buscar o post específico
const { data: post, pending } = await useAsyncData<Post>(`post-${slug}`, () => 
  queryContent('posts', slug).findOne()
)

// Buscar posts relacionados (mesmas tags)
const { data: relatedPosts } = await useAsyncData<Post[]>(`related-${slug}`, async () => {
  if (!post.value || !post.value.tags) return []
  
  const posts = await queryContent('posts')
    .where({ tags: { $contains: post.value.tags[0] } })
    .limit(2)
    .find()
  
  // Filtrar o post atual
  return posts.filter((p: Post) => p._path !== post.value?._path)
})

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

// Função para compartilhar post
const sharePost = async () => {
  if (navigator.share) {
    try {
      await navigator.share({
        title: post.value?.title || 'Post do blog',
        text: post.value?.description || '',
        url: window.location.href
      })
    } catch (error) {
      console.log('Erro ao compartilhar:', error)
    }
  } else {
    // Fallback para copiar URL
    try {
      await navigator.clipboard.writeText(window.location.href)
      alert('URL copiada para a área de transferência!')
    } catch (error) {
      console.log('Erro ao copiar URL:', error)
    }
  }
}

// Meta tags para SEO
useHead(() => ({
  title: post.value ? `${post.value.title} - Quila Dev` : 'Post não encontrado - Quila Dev',
  meta: [
    { name: 'description', content: post.value?.description || 'Post do blog Quila Dev' },
    { property: 'og:title', content: post.value?.title || 'Post do blog' },
    { property: 'og:description', content: post.value?.description || 'Post do blog Quila Dev' },
    { property: 'og:type', content: 'article' },
    { property: 'article:published_time', content: post.value?.date || '' },
    { property: 'article:author', content: post.value?.author || '' }
  ]
}))
</script>

<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Estilos para o conteúdo markdown */
:deep(.prose) {
  color: #374151;
}

:deep(.prose h1),
:deep(.prose h2),
:deep(.prose h3),
:deep(.prose h4),
:deep(.prose h5),
:deep(.prose h6) {
  color: #111827;
  font-weight: 700;
  margin-top: 2rem;
  margin-bottom: 1rem;
}

:deep(.prose h1) {
  font-size: 2.25rem;
  line-height: 2.5rem;
}

:deep(.prose h2) {
  font-size: 1.875rem;
  line-height: 2.25rem;
}

:deep(.prose h3) {
  font-size: 1.5rem;
  line-height: 2rem;
}

:deep(.prose p) {
  margin-bottom: 1.5rem;
  line-height: 1.75;
}

:deep(.prose ul),
:deep(.prose ol) {
  margin-bottom: 1.5rem;
  padding-left: 1.5rem;
}

:deep(.prose li) {
  margin-bottom: 0.5rem;
}

:deep(.prose code) {
  background-color: #f3f4f6;
  padding: 0.25rem 0.5rem;
  border-radius: 0.375rem;
  font-size: 0.875rem;
  font-weight: 600;
}

:deep(.prose pre) {
  background-color: #1f2937;
  color: #f9fafb;
  padding: 1rem;
  border-radius: 0.5rem;
  overflow-x: auto;
  margin-bottom: 1.5rem;
}

:deep(.prose pre code) {
  background-color: transparent;
  padding: 0;
  color: inherit;
}

:deep(.prose blockquote) {
  border-left: 4px solid #3b82f6;
  padding-left: 1rem;
  margin-left: 0;
  margin-right: 0;
  font-style: italic;
  color: #6b7280;
}

:deep(.prose a) {
  color: #3b82f6;
  text-decoration: underline;
}

:deep(.prose a:hover) {
  color: #1d4ed8;
}

:deep(.prose strong) {
  font-weight: 700;
  color: #111827;
}

:deep(.prose em) {
  font-style: italic;
}
</style>
