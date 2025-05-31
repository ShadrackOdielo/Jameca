<template>
<UCard :ui="{body:'flex ',header:'w-full'}" class="w-full ring-0 mx-auto mt-8 p-6 rounded-lg shadow-lg">
    <template #header>
        <h1 class="text-2xl font-bold mb-4">Add Product</h1>
    </template>
    <form @submit.prevent="handleSubmit">
        <div class="flex flex-cols-2 gap-6">
            <div class="flex flex-col">
            <UCard :ui="{body:'grid grid-cols-2 gap-2 ',header:'w-full ring-0'}" class="mb-6 ring-0">
                <template #header>
                    <h2 class="text-lg font-semibold">Basic Information</h2>
                </template>

                <!-- Existing fields -->
                <UFormField label="Product Name">
                    <UInput v-model="product.name" placeholder="Product Name" required />
                </UFormField>
                <UFormField label="Description">
                    <UInput v-model="product.description" placeholder="Description" type="textarea" />
                </UFormField>
                <UFormField label="Long Description">
                    <UTextarea v-model="product.long_description" placeholder="Long Description" type="textarea" />
                </UFormField>
                <UFormField label="Category">
                    <USelect v-model="product.category_id" :items="categories" placeholder="Select Category" required />
                </UFormField>
                <UFormField label="Brand">
                    <UInput v-model="product.brand" placeholder="Brand" required />
                </UFormField>
                <UFormField label="Is Active?">
                    <UCheckbox v-model="product.isActive">Is Active?</UCheckbox>
                </UFormField>
                <UFormField label="Is Featured?">
                    <UCheckbox v-model="product.isFeatured">Is Featured?</UCheckbox>
                </UFormField>
            </UCard>

            <!-- Variants Section -->
            <UCard :ui="{body:'flex flex-col gap-4',header:'ring-0'}" class="mb-6 ring-0">
                <template #header>
                    <h2 class="text-lg font-semibold">Product Variants</h2>
                </template>
                <div v-for="(variant, index) in product.variants" :key="index" class="flex items-center gap-4">
                    <UFormField label="Size">
                        <UInput v-model="variant.size" placeholder="Size (e.g., S, M, L, XL, 42)" />
                    </UFormField>
                    <UFormField label="Price ($)">
                        <UInput v-model="variant.price" placeholder="Price ($)" type="number" />
                    </UFormField>
                    <UFormField label="Stock Quantity">
                        <UInput v-model="variant.stock" placeholder="Stock Quantity" type="number" />
                    </UFormField>
                    <UButton
                        icon="i-lucide-trash-2"
                        variant="ghost"
                        class="text-red-500"
                        @click="removeVariant(index)"
                    />
                </div>
                <UButton
                    label="Add Variant"
                    icon="i-lucide-plus"
                    variant="outline"
                    @click="addVariant"
                />
            </UCard>
            </div>
            <!-- Images Upload Section -->
            <UCard :ui="{body:'flex flex-col gap-4',header:'ring-0'}" class="mb-6 ring-0">
                <template #header>
                    <h2 class="text-lg text-center font-semibold">Images Upload</h2>
                </template>
                <div class="flex flex-col items-center">
                    <UButton
                        label="Upload Images"
                        icon="i-lucide-upload"
                        variant="outline"
                        class="mb-4 h-46 w-46"
                        :class="{'cursor-pointer': !imageInput}"
                        @click="imageInput?.click()"
                    />
                    <input
                        ref="imageInput"
                        type="file"
                        multiple
                        accept="image/*"
                        class="hidden"
                        @change="handleImageChange"
                    />
                </div>
                <div class="mt-4 grid grid-cols-2 gap-4">
                    <div v-for="(img, index) in product.images" :key="index" class="relative">
                        <img :src="img" alt="Product Image" class="w-full h-32 object-cover rounded-md" />
                        <UButton
                            icon="i-lucide-trash-2"
                            variant="ghost"
                            class="absolute top-1 right-1 bg-white rounded-full p-1"
                            @click="product.images.splice(index, 1)"
                        />
                    </div>
                </div>
            </UCard>
        </div>
        <div class="flex justify-end space-x-2">
            <UButton type="submit" label="Create Product" icon="i-lucide-plus" />
            <UButton label="Cancel" variant="outline" @click="router.back()" />
        </div>
    </form>
</UCard>
</template>

<script setup lang="ts">

type Variant = {
    size: string | number;
    price: number;
    stock: number;
};

type Product = {
    id?: string;
    name: string;
    description: string;
    long_description: string;
    category_id: string;
    brand: string;
    images: string[];
    isActive: boolean;
    isFeatured: boolean;
    variants: Variant[];
};

const supabase = useSupabaseClient();
const product = ref<Product>({
    name: '',
    description: '',
    long_description: '',
    category_id: '',
    brand: '',
    images: [],
    isActive: true,
    isFeatured: false,
    variants: [],
});

const categories = ref<{ label: string; value: string }[]>([]);
const imageInput = ref<HTMLInputElement | null>(null);
const router = useRouter();
const toast = useToast();

const fetchCategories = async () => {
    const { data, error } = await supabase.from('categories').select('*');
    if (error) {
        console.error('Error fetching categories:', error);
    } else {
        categories.value = data.map((category) => ({
            label: category.name,
            value: category.id,
        }));
    }
};

const handleImageChange = async (e: Event) => {
    const target = e.target as HTMLInputElement;
    if (target.files) {
        const files = Array.from(target.files);
        for (const file of files) {
            const { data, error } = await supabase.storage
                .from('images')
                .upload(`products/${Date.now()}-${file.name}`, file);
            if (error) {
                console.error('Error uploading image:', error);
            } else {
                const publicUrl = supabase.storage.from('images').getPublicUrl(data.path).data.publicUrl;
                product.value.images.push(publicUrl);
            }
        }
    }
};

const addVariant = () => {
    product.value.variants.push({ size: '', price: 0, stock: 0 });
};

const removeVariant = (index: number) => {
    product.value.variants.splice(index, 1);
};

const handleSubmit = async () => {
    try {
        const { data: productData, error: productError } = await supabase
            .from('products')
            .insert({
                name: product.value.name,
                description: product.value.description,
                long_description: product.value.long_description,
                category_id: product.value.category_id,
                brand: product.value.brand,
                is_active: product.value.isActive,
                is_featured: product.value.isFeatured,
            })
            .select()
            .single();

        if (productError) throw productError;

        const productId = productData.id;

        // Insert images
        for (const imageUrl of product.value.images) {
            await supabase.from('productimages').insert({
                product_id: productId,
                image_url: imageUrl,
            });
        }

        // Insert variants
        for (const variant of product.value.variants) {
            await supabase.from('productvariants').insert({
                product_id: productId,
                size: String(variant.size),
                price: variant.price,
                stock: variant.stock,
            });
        }

        toast.add({
            title: 'Product Created',
            description: `Product ${product.value.name} has been created successfully.`,
            color: 'success',
        });
        await router.push('/products');
    } catch (err) {
        console.error('Error creating product:', err);
        toast.add({
            title: 'Error',
            description: 'Failed to create product. Please try again.',
            color: 'error',
        });
    }
};

// Fetch categories on component mount
onMounted(fetchCategories);
</script>