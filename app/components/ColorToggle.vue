<script setup>
const colorMode = useColorMode()

const isDark = computed({
  get() {
    return colorMode.value === 'dark'
  },
  set(_isDark) {
    colorMode.preference = _isDark ? 'dark' : 'light'
  }
})
</script>

<template>
  <ClientOnly v-if="!colorMode?.forced">
<USwitch
    v-model="isDark"
    checked-icon="i-lucide-moon"
    unchecked-icon="i-lucide-sun"
    color="secondary"
    class="text-2xl text-(--ui-secondary) hover:text-(--ui-secondary) transition-colors duration-300"
    />

    <template #fallback>
      <div class="size-8" />
    </template>
  </ClientOnly>
</template>
