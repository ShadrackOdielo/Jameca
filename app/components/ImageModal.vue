<script setup lang="ts">
const supabase = useSupabaseClient()
const {  productId } = defineProps<{
  productId: string
}>()

const emit = defineEmits<{ close: [boolean] }>()
const carousel = useTemplateRef('carousel')
const activeIndex = ref(0)

function onClickPrev() {
  activeIndex.value--
}
function onClickNext() {
  activeIndex.value++
}
function onSelect(index: number) {
  activeIndex.value = index
}

function select(index: number) {
  activeIndex.value = index

  carousel.value?.emblaApi?.scrollTo(index)
}
// fetch images from the productimages table
interface ProductImage {
  id: number
  product_id: string
  image_url: string
}
const productImages = ref<ProductImage[]>([])
const loading = ref(false)
const items = computed(() => {
  return productImages.value.map(img => img.image_url)
})
const fetchProductImages = async () => {
  loading.value = true
  const { data, error } = await supabase
    .from('productimages')
    .select('id, product_id, image_url')
    .eq('product_id', productId)

  if (error) {
    console.error('Error fetching product images:', error)
  } else {
    productImages.value = data?.map(img => ({
      id: Number(img.id),
      product_id: img.product_id,
      image_url: img.image_url
    })) || []
  }
  loading.value = false
}
onMounted(() => {
  fetchProductImages()
})

</script>

<template>
  <UModal
    :close="{ onClick: () => emit('close', false) }"
    :ui="{body:'max-w-2xl'}"
    :title="'Product Images'">
  <template #body>
  <div class="flex-1 w-full">
    <UCarousel
      ref="carousel"
      v-slot="{ item }"
      arrows
      :items="items"
      :prev="{ onClick: onClickPrev }"
      :next="{ onClick: onClickNext }"
      class="w-full max-w-xs mx-auto"
      @select="onSelect"
    >
      <img :src="item" width="320" height="320" class="rounded-lg">
    </UCarousel>

    <div class="flex gap-1 justify-between pt-4 max-w-xs mx-auto">
      <div
        v-for="(item, index) in items"
        :key="index"
        class="size-11 opacity-25 hover:opacity-100 transition-opacity"
        :class="{ 'opacity-100': activeIndex === index }"
        @click="select(index)"
      >
        <img :src="item" width="44" height="44" class="rounded-lg">
      </div>
    </div>
  </div>
  </template>
  </UModal>
</template>
