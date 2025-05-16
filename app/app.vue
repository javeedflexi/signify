<script setup lang="ts">
useScriptPlausibleAnalytics({
  domain: 'inkly.hrcd.fr',
  scriptInput: {
    src: 'https://analytics.hrcd.fr/js/script.js',
  }
})

useSeoMeta({
  title: 'FlexiFunnels - Email Signature Generator',
  description: 'The most simple open-source email signature generator',
  author: 'Hugo Richard',
  twitterCreator: '@HugoRCD__',
  twitterTitle: 'FlexiFunnels - Email Signature Generator',
  twitterDescription: 'The most simple open-source email signature generator',
  twitterCard: 'summary_large_image',
  twitterImage: 'https://inkly.hrcd.fr/og.png',
  ogUrl: 'https://inkly.hrcd.fr',
  ogImage: 'https://inkly.hrcd.fr/og.png',
  ogSiteName: 'FlexiFunnels - Email Signature Generator',
  ogTitle: 'FlexiFunnels - Email Signature Generator',
  ogDescription: 'The most simple open-source email signature generator',
})

useHead({
  link: [
    {
      rel: 'icon',
      type: 'image/svg+xml',
      href: '/flexifunnels-logo.svg',
    },
    {
      rel: 'stylesheet',
      href: 'https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap',
    }
  ]
})
</script>

<template>
  <Html lang="en">
    <UApp>
      <div class="flex h-full flex-col bg-white">
        <LayoutNavbar />
        <div class="h-full">
          <NuxtPage />
        </div>
      </div>
    </UApp>
  </Html>
</template>

<style>
#__nuxt, #__layout, body, html {
  height: 100%;
  font-family: 'Inter', sans-serif;
  background-color: #ffffff;
  color: #333333;
}

:root {
  --primary: #22c55e;
  --primary-hover: #16a34a;
}

.text-primary {
  color: var(--primary);
}

.bg-primary {
  background-color: var(--primary);
}

.border-primary {
  border-color: var(--primary);
}

.btn-primary {
  background-color: var(--primary);
  color: white;
  transition: background-color 0.2s;
}

.btn-primary:hover {
  background-color: var(--primary-hover);
}

/* Override Nuxt UI form styling */
.u-input {
  background-color: #ffffff;
  border-color: #e5e7eb;
}

.u-input:focus, .u-input:focus-within {
  border-color: var(--primary);
  box-shadow: 0 0 0 1px var(--primary);
}

.u-button-primary {
  background-color: var(--primary);
  color: white;
}

.u-button-primary:hover {
  background-color: var(--primary-hover);
}
</style>
