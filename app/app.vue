<template>
  <div>
    <NuxtRouteAnnouncer />
    <header class="bg-neutral-900 text-neutral-400">
      <div class="container mx-auto py-4">
        <div class="flex justify-between items-center">
          <nuxt-link
            to="/"
            class="inline-block w-[200px] h-10 bg-neutral-950 rounded-full"
          ></nuxt-link>
          <div class="flex justify-end space-x-10">
            <nuxt-link class="hover:text-white" to="/">Início</nuxt-link>
            <nuxt-link class="hover:text-white" to="/posts">Posts</nuxt-link>
          </div>
        </div>
      </div>
    </header>
    <NuxtPage />
  </div>
</template>
