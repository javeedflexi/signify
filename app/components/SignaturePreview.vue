<script setup lang="ts">
import type { Signature } from '~~/types'

const props = defineProps<Signature & { theme?: string }>()
const { data, options, theme = 'dark' } = props

const signatureContainer = ref<HTMLElement>()
const cleanSignatureRef = ref<HTMLElement>()
const toast = useToast()

const { copy, copied } = useClipboard({ 
  // Instead of directly copying the inner HTML, we'll use a cleaned version
  source: () => {
    if (!signatureContainer.value) return ''
    
    // Create a clone of the signature container
    const clone = signatureContainer.value.cloneNode(true) as HTMLElement
    
    // Process the clone to convert dynamic bindings to static attributes
    const processElement = (element: HTMLElement) => {
      // Remove Vue-specific attributes
      const attrs = [...element.attributes]
      attrs.forEach(attr => {
        if (attr.name.startsWith('v-') || attr.name.startsWith('@')) {
          element.removeAttribute(attr.name)
        }
        // Convert bound styles to inline styles
        else if (attr.name === ':style' || attr.name === 'v-bind:style') {
          element.removeAttribute(attr.name)
        }
      })
      
      // Process all child elements
      Array.from(element.children).forEach(child => {
        processElement(child as HTMLElement)
      })
    }
    
    processElement(clone)
    
    // Process specific elements to ensure they have proper inline styles
    const imgElements = clone.querySelectorAll('img')
    imgElements.forEach(img => {
      if (options.image.form === 'circle') {
        img.style.borderRadius = `${options.image.size}px`
        img.style.width = `${options.image.size}px`
        img.style.height = `${options.image.size}px`
      } else if (options.image.form === 'square') {
        img.style.width = `${options.image.size}px`
        img.style.height = `${options.image.size}px`
      } else {
        img.style.width = `${options.image.size}px`
      }
      
      img.style.objectFit = 'cover'
      
      if (options.image.border) {
        img.style.border = `${options.image.borderWidth}px ${options.image.borderStyle} ${options.image.borderColor}`
      }
      
      if (options.image.shadow) {
        img.style.boxShadow = `0 ${options.image.shadowIntensity * 2}px ${options.image.shadowIntensity * 4}px -${options.image.shadowIntensity}px rgba(0, 0, 0, ${options.image.shadowIntensity * 0.05})`
      }
    })
    
    // Apply cell vertical alignment
    const tdElements = clone.querySelectorAll('td')
    if (tdElements.length > 0) {
      const firstCell = tdElements[0]
      if (options.image.align === 'top') {
        firstCell.style.verticalAlign = 'top'
      } else if (options.image.align === 'center') {
        firstCell.style.verticalAlign = 'middle'
      } else if (options.image.align === 'bottom') {
        firstCell.style.verticalAlign = 'bottom'
      }
    }
    
    // Apply font styling to name
    const nameElement = clone.querySelector('table table tr:first-child td')
    if (nameElement) {
      nameElement.style.fontSize = `${options.size.title}px`
      nameElement.style.color = options.color.title
      nameElement.style.fontWeight = options.font.titleWeight
    }
    
    // Apply styling to social links
    const socialLinks = clone.querySelectorAll('a')
    socialLinks.forEach(link => {
      link.style.color = '#22c55e'
      link.style.textDecoration = 'underline'
    })
    
    // Return the cleaned HTML
    return clone.innerHTML
  },
  copiedDuring: 1000 
})

function copyToClipboard() {
  try {
    // Create a function that manually copies cleaned HTML to clipboard
    const signatureElement = signatureContainer.value
    if (!signatureElement) return
    
    // Create a clean copy
    const tempContainer = document.createElement('div')
    tempContainer.innerHTML = signatureElement.innerHTML
    
    // Remove Vue-specific attributes
    const allElements = tempContainer.querySelectorAll('*')
    allElements.forEach(el => {
      // Remove Vue attributes
      for (let i = 0; i < el.attributes.length; i++) {
        const attr = el.attributes[i]
        if (attr.name.startsWith('v-') || attr.name.startsWith(':') || attr.name.startsWith('@')) {
          el.removeAttribute(attr.name)
          i--
        }
      }
      
      // Make tables and cells transparent
      if (el.tagName === 'TABLE' || el.tagName === 'TR' || el.tagName === 'TD' || el.tagName === 'TBODY') {
        el.style.backgroundColor = 'transparent'
        el.style.background = 'transparent'
      }
    })
    
    // Manual copy using the document.execCommand
    const selection = window.getSelection()
    const range = document.createRange()
    
    // Create a temporary div to hold our HTML
    const tempDiv = document.createElement('div')
    tempDiv.style.position = 'absolute'
    tempDiv.style.left = '-9999px'
    tempDiv.innerHTML = tempContainer.innerHTML
    document.body.appendChild(tempDiv)
    
    // Select and copy
    range.selectNodeContents(tempDiv)
    selection.removeAllRanges()
    selection.addRange(range)
    
    const success = document.execCommand('copy')
    
    // Clean up
    document.body.removeChild(tempDiv)
    selection.removeAllRanges()
    
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

defineExpose({
  copyToClipboard
})
</script>

<template>
  <div class="w-full">
    <div ref="signatureContainer" class="py-2" id="signaturePreview">
      <ClientOnly>
        <table style="width: 100%; background: transparent; background-color: transparent;" :style="options.color.transparent ? {backgroundColor: 'transparent'} : { backgroundColor: `${options.color.background}` }">
          <tbody>
            <tr>
              <td 
                style="padding: 6px; background: transparent;" 
                :style="[
                  { width: `${options.image.size + options.gap.image}px` },
                  options.image.align === 'top' ? { verticalAlign: 'top' } : {},
                  options.image.align === 'center' ? { verticalAlign: 'middle' } : {},
                  options.image.align === 'bottom' ? { verticalAlign: 'bottom' } : {},
                ]"
              >
                <img
                  :src="data.image"
                  alt="Profile Picture"
                  :style="[
                    options.image.form === 'rectangle' ? { width: `${options.image.size}px` } : {},
                    options.image.form === 'square' ? { width: `${options.image.size}px`, height: `${options.image.size}px` } : {},
                    options.image.form === 'circle' ? { width: `${options.image.size}px`, height: `${options.image.size}px`, borderRadius: `${options.image.size}px` } : {},
                    { objectFit: 'cover' },
                    options.image.border ? {
                      border: `${options.image.borderWidth}px ${options.image.borderStyle} ${options.image.borderColor}`,
                    } : {},
                    options.image.shadow ? {
                      boxShadow: `0 ${options.image.shadowIntensity * 2}px ${options.image.shadowIntensity * 4}px -${options.image.shadowIntensity}px rgb(0 0 0 / ${options.image.shadowIntensity * 0.05})`
                    } : {},
                  ]"
                >
              </td>
              <td
                style="padding: 6px; background: transparent;"
                :style="[
                  { fontSize: `${options.size.subtitle}px` }
                ]"
                :class="[
                  options.font.family === 'inter' ? 'font-inter' :
                  options.font.family === 'sf' ? 'font-sf' :
                  options.font.family === 'roboto' ? 'font-roboto' :
                  'font-arial'
                ]"
              >
                <table style="font-size: 14px; background: transparent; background-color: transparent;" :style="{ fontSize: `${options.size.subtitle}px` }">
                  <tr v-if="data.fullName">
                    <td
                      :style="[
                        { 
                          fontSize: `${options.size.title}px`,
                          color: options.color.title,
                          fontWeight: options.font.titleWeight
                        }
                      ]"
                    >
                      {{ data.fullName }}
                    </td>
                  </tr>
                  <tr v-if="data.jobTitle" :style=" { color: `${options.color.subtitle}`}">
                    <td>
                      {{ data.jobTitle }}
                    </td>
                  </tr>
                  <tr v-if="data.company" :style=" { color: `${options.color.subtitle}`}">
                    <td :style="{ fontSize: `${options.size.subtitle}px`, color: `${options.color.subtitle}` }">
                      {{ data.company }}
                    </td>
                  </tr>
                  <tr v-if="data.email" :style=" { color: `${options.color.subtitle}`}">
                    <td :style="{ fontSize: `${options.size.subtitle}px`, color: `${options.color.subtitle}` }">
                      {{ data.email }}
                    </td>
                  </tr>
                  <tr v-if="data.phone" :style=" { color: `${options.color.subtitle}`}">
                    <td>
                      {{ data.phone }}
                    </td>
                  </tr>
                  <tr v-if="data.socials.length > 0">
                    <td>
                      <table style="background: transparent; background-color: transparent;">
                        <tbody>
                          <tr :style="{ fontSize: `${options.size.social}px` }">
                            <td v-for="social in data.socials.filter((social) => social.url)" :key="social.title" :style="{ paddingRight: `${options.gap.social}px` }">
                              <a :href="social.url" style="text-decoration: underline" :style="{ color: '#22c55e' }">{{ social.title }}</a>
                            </td>
                          </tr>
                        </tbody>
                      </table>
                    </td>
                  </tr>
                </table>
              </td>
            </tr>
          </tbody>
        </table>
        <template #fallback>
          <div class="h-24 animate-pulse bg-slate-700 rounded-md" />
        </template>
      </ClientOnly>
    </div>

    <!-- Button is now provided by the parent component -->
  </div>
</template>

