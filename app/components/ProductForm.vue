<template>
<UCard :ui="{body:'flex ',header:'w-full'}" class="w-full ring-0 mx-auto mt-8 p-6 rounded-lg shadow-lg">
    <template #header>
        <h1 class="text-2xl font-bold mb-4">Add Product</h1>
    </template>
    <form @submit.prevent="handleSubmit">
        <div class="flex flex-row md:flex-cols-2 gap-6">
            <div class="flex flex-col">
                <!-- Existing fields -->
                <UCard :ui="{body:'grid grid-cols-2 gap-2 ',header:'w-full ring-0'}" class="mb-6 ring-0">
                    <template #header>
                        <h2 class="text-lg font-semibold">Basic Information</h2>
                    </template>
                    <UFormField label="Product Name">
                        <UInput v-model="product.name" placeholder="Product Name" required />
                    </UFormField>
                    <UFormField label="Category">
                        <USelect v-model="product.category_id" :items="categories" placeholder="Select Category" required />
                    </UFormField>
                    <UFormField label="Description" class="col-span-2 w-full" >
                        <UInput v-model="product.description" placeholder="Description" />
                    </UFormField>
                    <UFormField label="Long Description" class="col-span-2 w-full">
                        <UTextarea v-model="product.long_description" placeholder="Long Description"  />
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

                    <UFormField label="Variants" class="col-span-2">
                        <p class="text-sm text-gray-500 mb-2">Add product variants (size can be S,M,L or 22,34 ...)</p>
                        <div class="flex flex-col gap-2">
                            <div v-for="(variant, index) in product.variants" :key="index" class="flex items-center gap-2">
                                <UInput v-model="variant.size" placeholder="Size" class="w-1/3" />
                                <UInput v-model.number="variant.price" placeholder="Price" type="number" class="w-1/3" />
                                <UInput v-model.number="variant.stock" placeholder="Stock" type="number" class="w-1/3" />
                                <UButton
                                    icon="i-lucide-trash-2"
                                    variant="ghost"
                                    class="ml-2"
                                    @click="removeVariant(index)"
                                />
                            </div>
                            <UButton
                                label="Add Variant"
                                icon="i-lucide-plus"
                                variant="outline"
                                class="mt-2"
                                @click="addVariant"
                            />
                        </div>
                    </UFormField>
                </UCard>
            </div>
            <!-- Images Upload Section -->
            <UCard :ui="{body:'flex flex-col gap-4',header:'ring-0'}" class="mb-6 ring-0 ">
                <template #header>
                    <h2 class="text-lg text-center font-semibold">Images Upload</h2>
                </template>
                <div class="flex flex-col items-center ">
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
                    >
                </div>
                <UButton
                    v-if="imagePreviews.length > 0"
                    label="Clear Images"
                    icon="i-lucide-trash-2"
                    variant="ghost"
                    class="mb-4"
                    @click="imagePreviews = []"
                />
                <div class=" flex flex-col gap-2">
                    <div v-for="(img, index) in imagePreviews" :key="index" class="relative  flex  flex-row gap-4">
                        <img :src="img.previewUrl" alt="Image Preview" class="w-auto h-8 object-cover rounded-md" />
                        <div class="flex flex-col">
                            <span class="text-sm font-semibold">{{ img.file.name }}</span>
                            <span class="text-xs text-gray-500">{{ (img.file.size / 1024).toFixed(2) }} KB</span>
                        </div>
                        <UButton
                            icon="i-lucide-trash-2"
                            variant="ghost"
                            @click="imagePreviews.splice(index, 1)"
                        />
                    </div>
                </div>
            </UCard>
        </div>
        <div class="w-full flex-none justify-between space-x-2">
            <UButton type="submit" label="Submit Form" />
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
    images: string[]; // URLs of uploaded images
    isActive: boolean;
    isFeatured: boolean;
    variants: Variant[];
};

type ImagePreview = {
    file: File;
    previewUrl: string;
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
const imagePreviews = ref<ImagePreview[]>([]); // Store image previews
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

const handleImageChange = (e: Event) => {
    const target = e.target as HTMLInputElement;
    if (target.files) {
        const files = Array.from(target.files);
        for (const file of files) {
            const previewUrl = URL.createObjectURL(file);
            imagePreviews.value.push({ file, previewUrl });
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
        // Upload images
        const uploadedImageUrls: string[] = [];
        for (const imagePreview of imagePreviews.value) {
            const { data, error } = await supabase.storage
                .from('images')
                .upload(`products/${Date.now()}-${imagePreview.file.name}`, imagePreview.file);
            if (error) {
                console.error('Error uploading image:', error);
                throw error;
            }
            const publicUrl = supabase.storage.from('images').getPublicUrl(data.path).data.publicUrl;
            uploadedImageUrls.push(publicUrl);
        }

        // Add uploaded image URLs to the product
        product.value.images = uploadedImageUrls;

        // Insert product data
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