<script setup lang="ts">
import type { SignatureFormData, SignatureOptions } from '~~/types'
import { ref, onMounted } from 'vue'


// Define a type for the signaturePreview ref that includes the copyToClipboard method
interface SignaturePreviewRef {
  copyToClipboard: () => void
}

const signaturePreview = ref<SignaturePreviewRef | null>(null)
const toast = useToast()

// Function to copy the signature to clipboard
function copySignature() {
  const signaturePreviewRef = document.querySelector('#signaturePreview')
  if (!signaturePreviewRef) return
  
  try {
    // Create a temporary container to hold the signature HTML
    const tempContainer = document.createElement('div')
    tempContainer.innerHTML = signaturePreviewRef.innerHTML
    
    // Clean up Vue-specific attributes
    const vueAttrs = tempContainer.querySelectorAll('*')
    vueAttrs.forEach(el => {
      for (let i = 0; i < el.attributes.length; i++) {
        const attr = el.attributes[i]
        if (attr.name.startsWith('v-') || attr.name.startsWith(':') || attr.name.startsWith('@')) {
          el.removeAttribute(attr.name)
          i--
        }
      }
      
      // Ensure all elements don't have a background
      if (el.tagName === 'TABLE' || el.tagName === 'TD' || el.tagName === 'TR' || el.tagName === 'TBODY') {
        el.style.backgroundColor = 'transparent'
        el.style.background = 'transparent'
      }
    })
    
    // Make background transparent for all tables, including nested ones
    const tables = tempContainer.querySelectorAll('table, td, tr, tbody')
    tables.forEach(el => {
      // Remove background color from tables and cells
      el.style.backgroundColor = 'transparent'
      el.style.background = 'transparent'
      
      // Remove any background-related attributes
      el.removeAttribute('bgcolor')
    })
    
    // Create a selection and range
    const selection = window.getSelection()
    const range = document.createRange()
    
    // Clear any existing selection
    selection.removeAllRanges()
    
    // Put the cleaned HTML in a temporary div and select it
    const tempDiv = document.createElement('div')
    tempDiv.style.position = 'absolute'
    tempDiv.style.left = '-9999px'
    tempDiv.innerHTML = tempContainer.innerHTML
    document.body.appendChild(tempDiv)
    
    range.selectNodeContents(tempDiv)
    selection.addRange(range)
    
    // Execute the copy command
    const success = document.execCommand('copy')
    
    // Clean up
    document.body.removeChild(tempDiv)
    selection.removeAllRanges()
    
    // Show success/failure notification
    if (success) {
      toast.add({
        title: 'Signature copied',
        description: 'Your signature has been copied to clipboard.',
        icon: 'i-lucide-check-circle',
        color: 'green',
        timeout: 2000
      })
    } else {
      toast.add({
        title: 'Copy failed',
        description: 'Failed to copy signature. Please try an alternative method.',
        icon: 'i-lucide-alert-circle',
        color: 'red',
        timeout: 3000
      })
    }
  } catch (error) {
    toast.add({
      title: 'Copy failed',
      description: 'Failed to copy signature. Please try an alternative method.',
      icon: 'i-lucide-alert-circle',
      color: 'red',
      timeout: 3000
    })
  }
}

const options = ref<SignatureOptions>({
  gap: {
    title: 10,
    subtitle: 10,
    social: 10,
    image: 10,
  },
  size: {
    title: 16,
    subtitle: 14,
    social: 14,
  },
  color: {
    title: '#22c55e',
    autoTitle: false,
    subtitle: '#4B5563',
    social: '#22c55e',
    background: '#ffffff',
    transparent: true,
  },
  image: {
    align: 'center',
    form: 'circle',
    size: 90,
    border: true,
    borderStyle: 'solid',
    borderColor: '#22c55e',
    borderWidth: 2,
    shadow: true,
    shadowIntensity: 2
  },
  font: {
    family: 'inter',
    titleWeight: '600'
  }
})

const data = ref<SignatureFormData>({
  image: 'https://avatars.githubusercontent.com/u/71938701?v=4',
  fullName: 'John',
  jobTitle: 'AI Specialist at FlexiFunnels',
  company: '',
  email: '',
  phone: '(+33) 6 00 00 00 00',
  socials: [
    {
      title: 'Website',
      url: '',
      type: '',
    },
    {
      title: 'Twitter',
      url: '',
      type: 'twitter',
    },
    {
      title: 'Instagram',
      url: '',
      type: 'instagram',
    },
    {
      title: 'GitHub',
      url: '',
      type: 'github',
    },
  ],
})

const theme = ref('dark')

const emailSubject = 'Professionnal Email Signature'
const emailContent = ref(`Hello,

I'm pleased to share with you my new email signature created with FlexiFunnels.
Feel free to reach out if you have any questions!

Best regards,`)

const isMobile = ref(false)
const showSidebar = ref(true)

onMounted(() => {
  const checkMobile = () => {
    isMobile.value = window.innerWidth < 1024
    if (isMobile.value) {
      showSidebar.value = false
    } else {
      showSidebar.value = true
    }
  }
  
  checkMobile()
  window.addEventListener('resize', checkMobile)
  
  return () => {
    window.removeEventListener('resize', checkMobile)
  }
})
</script>

<template>
  <div class="flex h-full flex-col lg:flex-row">
    <div 
      v-if="showSidebar" 
      class="w-full lg:w-80 h-full overflow-y-auto border-r border-gray-200 bg-white flex flex-col"
    >
      <div class="sticky top-0 z-10 px-4 py-3 border-b border-gray-200 flex items-center justify-between">
        <h2 class="text-sm font-medium text-gray-700">
          Signature Settings
        </h2>
        <UButton 
          icon="i-lucide-x" 
          color="gray" 
          variant="ghost" 
          class="lg:hidden"
          size="xs"
          @click="showSidebar = false"
        />
      </div>
      <div class="p-4">
        <SettingsOption v-model:options="options" v-model:data="data" />
      </div>
    </div>

    <div class="lg:hidden fixed bottom-4 right-4 z-20">
      <UButton 
        v-if="!showSidebar"
        icon="i-lucide-settings"
        color="green" 
        variant="solid"
        size="lg"
        class="shadow-lg" 
        @click="showSidebar = true"
      />
    </div>

    <div class="relative flex-1 h-full overflow-auto bg-white">
      <div class="absolute right-4 top-4 z-10 flex gap-2">
        <UTooltip text="Email preview mode">
          <UButton
            icon="i-lucide-mail"
            color="gray"
            variant="ghost"
            size="xs"
            disabled
          />
        </UTooltip>
      </div>

      <div class="h-full p-4 flex flex-col">
        <div class="w-full max-w-3xl mx-auto bg-white h-full flex flex-col overflow-hidden border border-gray-200 rounded-lg shadow-sm">
          <div class="bg-gray-50 px-4 py-3 border-b border-gray-200">
            <div class="flex flex-col gap-2">
              <div class="flex justify-between items-center">
                <h3 class="font-medium text-sm text-gray-700">
                  {{ emailSubject }}
                </h3>
                <div class="flex gap-1">
                  <UIcon name="i-lucide-archive" class="text-gray-500 text-sm" />
                  <UIcon name="i-lucide-trash" class="text-gray-500 text-sm" />
                  <UIcon name="i-lucide-reply" class="text-gray-500 text-sm" />
                </div>
              </div>
              <div class="flex items-center gap-2">
                <UAvatar
                  :src="data.image"
                  :alt="data.fullName"
                  size="xs"
                />
                <div class="text-xs">
                  <span class="font-medium text-gray-700">{{ data.fullName }}</span>
                  <span class="text-gray-500"> ({{ data.email || 'email@example.com' }})</span>
                </div>
              </div>
            </div>
          </div>
          
          <div class="p-5 flex-1 overflow-auto bg-white text-gray-700">
            <div class="max-w-2xl space-y-6">
              <div class="whitespace-pre-line text-sm text-gray-600">
                {{ emailContent }}
              </div>

              <SignaturePreview
                ref="signaturePreview"
                :data
                :options
                :theme="'light'"
              />

              <div class="flex justify-center">
                <UButton
                  label="Copy Signature"
                  color="green"
                  variant="solid"
                  class="mt-4 font-medium"
                  icon="i-lucide-copy"
                  @click="copySignature()"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

