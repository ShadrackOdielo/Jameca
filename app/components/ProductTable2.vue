<script setup lang="ts">
import type { TableColumn, DropdownMenuItem } from '@nuxt/ui'
import type { Database } from '~/types/database.types'
import { LazyImageModal } from '#components'


const toast = useToast()
const overlay = useOverlay()

const modal = overlay.create(LazyImageModal, {
  props: {
    productId: ''
  }
})

async function open(productId: string) {
   modal.open()
  modal.patch({
    productId: productId
  })
}

 
 

interface Product  {
  id: string
  uuid: string
  name: string
  description: string
  price: number
  stock: number
  brand: string
  category_name: string
  is_active: boolean
  is_featured: boolean
  created_at: string
  image_url: string | null
}

const supabase = useSupabaseClient<Database>()
const products = ref<Product[]>([])
const loading = ref(false)

const fetchProducts = async () => {
  loading.value = true
  const { data, error } = await supabase
    .from('products')
    .select(`
      uuid:id, 
      name, 
      description, 
      brand, 
      is_active, 
      is_featured, 
      created_at, 
      categories (name), 
      productimages (image_url),
      productvariants (id, price, stock, size)
    `)

  if (error) {
    console.error('Error fetching products:', error)
  } else {
    // Flatten product variants into unique inventory items
    products.value = (data || []).flatMap((product, index) =>
      (product.productvariants || []).map((variant) => ({
        id: `${index + 1}-${variant.id}`, // Unique ID combining product and variant
        uuid: product.uuid,
        name: product.name,
        description: product.description ?? '',         // Default empty string
        brand: product.brand,
        category_name: product.categories.name,
        is_active: product.is_active ?? false,          // Default false
        is_featured: product.is_featured ?? false,      // Default false
        created_at: product.created_at ?? '',           // Default empty string
        image_url: product.productimages?.[0]?.image_url || null, // First image or null
        price: variant.price,                           // Variant price
        stock: variant.stock,                           // Variant stock
        size: variant.size || 'N/A'                    // Variant size or default 'N/A'
      }))
    )
  }
  loading.value = false
}



const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'id',
    header: 'ID'
  },
  {
    accessorKey: 'name',
    header: 'Name'
  },
  {
    accessorKey: 'price',
    header: 'Price',
  },
  {
    accessorKey: 'size',
    header: 'Size',
  },
  {
    accessorKey: 'stock',
    header: 'Stock',
  },
  {
    accessorKey: 'description',
    header: 'Description'
  },
  {
    accessorKey: 'created_at',
    header: 'Created At',
    cell: ({ row }) =>
      new Date(row.original.created_at || '').toLocaleDateString()
  },
  {
    accessorKey: 'is_active',
    header: 'Active',
    cell: ({ row }) => (row.original.is_active ? 'Yes' : 'No')
  },
  {
    accessorKey: 'is_featured',
    header: 'Featured',
    cell: ({ row }) => (row.original.is_featured ? 'Yes' : 'No')
  },
  
  {
    id: 'action'
  }
]

function getDropdownActions(product: Product): DropdownMenuItem[][] {
  return [
    [
      {
        label: 'Copy user Id',
        icon: 'i-lucide-copy',
        onSelect: () => {
          navigator.clipboard.writeText(product.id.toString())
          toast.add({
            title: 'User ID copied to clipboard!',
            color: 'success',
            icon: 'i-lucide-circle-check'
          })
        }
      }
    ],
    [
      {
        label: 'Edit',
        icon: 'i-lucide-edit'
      },
      {
        label: 'Delete',
        icon: 'i-lucide-trash',
        color: 'error'
      }
    ]
  ]
}
onMounted(fetchProducts)

</script>

<template>
  <div class="w-full">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-xl font-bold">Product List</h2>
      <UButton icon="i-lucide-refresh-cw" :loading="loading" @click="fetchProducts">
        Refresh
      </UButton>
        <UButton icon="i-lucide-plus" to="/products/create" color="primary">
            Add Product
        </UButton>
    </div>

    <UCard>
  <UTable :data="products" :columns="columns" class="flex-1">
    <template #name-cell="{ row }">
      <div class="flex items-center gap-3">
        
        <UAvatar
          :src="`${row.original.image_url || '/logobonsus.png'}`"
          :fallback="row.original.name.charAt(0).toUpperCase()"
          size="lg"
          :alt="`${row.original.name} avatar`"
          @click="open(row.original.uuid)"
        />
        <div>
          <p class="font-medium text-highlighted">
            {{ row.original.name }}
          </p>
          <p>
            {{ row.original.category_name }}
          </p>
        </div>
      </div>
    </template>
    <template #action-cell="{ row }">
      <UDropdownMenu :items="getDropdownActions(row.original)">
        <UButton
          icon="i-lucide-ellipsis-vertical"
          color="neutral"
          variant="ghost"
          aria-label="Actions"
        />
      </UDropdownMenu>
    </template>
  </UTable>
    </UCard>

  </div>
</template>
