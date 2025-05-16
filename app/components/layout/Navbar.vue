<script setup lang="ts">
const header = ref(null)
const { height } = useElementSize(header)

onMounted(() => {
  document.documentElement.style.setProperty('--header-height', `${height.value}px`)
})
</script>

<template>
  <header
    ref="header"
    class="px-4 py-3 flex items-center justify-between border-b border-slate-800 bg-slate-900"
  >
    <div class="flex items-center">
      <div class="flex flex-col">
        <h1 class="text-2xl font-bold">
          <span class="text-primary">Flexi</span><span class="text-white">Funnels</span>
        </h1>
        <p class="text-xs text-gray-400">Email sign. generator</p>
      </div>
    </div>
  </header>
</template>
