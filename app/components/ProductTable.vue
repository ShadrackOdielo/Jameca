<script setup lang="ts">
import { ref, onMounted } from 'vue'
import type { TableColumn } from '@nuxt/ui'
import type { Database } from '~/types/database.types'

type Product = {
  id: number
  uuid: string
  name: string
  description: string
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
      productimages (image_url)
    `)

  if (error) {
    console.error('Error fetching products:', error)
  } else {
    products.value = (data || []).map((product, index) => ({
      id: index + 1, // Sequential ID
      uuid: product.uuid,
      name: product.name,
      description: product.description ?? '',         // default empty string
      brand: product.brand,
      category_name: product.categories.name,
      is_active: product.is_active ?? false,          // default false
      is_featured: product.is_featured ?? false,      // default false
      created_at: product.created_at ?? '',           // default empty string
      image_url: product.productimages?.[0]?.image_url || null // First image or null
    }))
  }
  loading.value = false
}

const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'id',
    header: 'ID',
    cell: ({ row }) => row.original.id
  },
  {
    accessorKey: 'image_url',
    header: 'Image',
    cell: ({ row }) =>
      row.original.image_url
        ? h('img', {
            src: row.original.image_url,
            alt: row.original.name,
            class: 'w-16 h-16 object-cover rounded-md'
          })
        : 'No Image'
  },
  {
    accessorKey: 'name',
    header: 'Name',
    cell: ({ row }) => row.original.name
  },
  {
    accessorKey: 'brand',
    header: 'Brand',
    cell: ({ row }) => row.original.brand
  },
  {
    accessorKey: 'category_name',
    header: 'Category',
    cell: ({ row }) => row.original.category_name
  },
  {
    accessorKey: 'is_active',
    header: 'Status',
    cell: ({ row }) =>
      row.original.is_active
        ? 'Active'
        : 'Inactive'
  },
  {
    accessorKey: 'is_featured',
    header: 'Featured',
    cell: ({ row }) =>
      row.original.is_featured
        ? 'Yes'
        : 'No'
  },
  {
    accessorKey: 'created_at',
    header: 'Created At',
    cell: ({ row }) =>
      new Date(row.original.created_at || '').toLocaleDateString()
  },
  {
    id: 'actions',
    header: 'Actions',
    cell: ({ row }) => h('div', { class: 'flex space-x-2' }, [
      h('UButton', {
        icon: 'i-lucide-eye',
        color: 'secondary',
        variant: 'ghost',
        onClick: () => viewProduct(row.original.uuid)
      }),
      h('UButton', {
        icon: 'i-lucide-pencil',
        color: 'primary',
        variant: 'ghost',
        onClick: () => editProduct(row.original.uuid)
      }),
      h('UButton', {
        icon: 'i-lucide-trash-2',
        color: 'error',
        variant: 'ghost',
        onClick: () => deleteProduct(row.original.uuid)
      })
    ])
  }
]

const viewProduct = (uuid: string) => {
  console.log(`View product with UUID: ${uuid}`)
}

const editProduct = (uuid: string) => {
  console.log(`Edit product with UUID: ${uuid}`)
}

const deleteProduct = async (uuid: string) => {
  if (confirm('Are you sure you want to delete this product?')) {
    const { error } = await supabase.from('products').delete().eq('uuid', uuid)
    if (error) {
      console.error('Error deleting product:', error)
    } else {
      products.value = products.value.filter((product) => product.uuid !== uuid)
    }
  }
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
      <UTable
        :data="products"
        :columns="columns"
        :loading="loading"
        hover
        class="min-h-[500px]"
      >
        <template #empty-state>
          <div class="flex flex-col items-center justify-center py-6">
            <UIcon name="i-lucide-package" class="text-gray-400 mb-2" size="xl" />
            <p class="text-sm text-gray-500 dark:text-gray-400">No products found</p>
          </div>
        </template>
      </UTable>
    </UCard>
  </div>
</template>
