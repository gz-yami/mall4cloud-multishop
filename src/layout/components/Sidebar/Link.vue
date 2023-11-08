<template>
  <component
    :is="type"
    v-bind="linkProps(to)"
  >
    <slot />
  </component>
</template>

<script setup>
import { isExternal } from '@/utils/validate'

const props = defineProps({
  to: {
    type: String,
    required: true
  }
})
  computed: {
    isExternal () {
      return isExternal(to)
    },
    type () {
      if (isExternal) {
        return 'a'
      }
      return 'router-link'
    }
  },

const linkProps  = (to) => {
  if (isExternal) {
    return {
      href: to,
      target: '_blank',
      rel: 'noopener'
    }
  }
  return {
    to
  }
}

</script>
