<template>
    <nav class="w-full top-0 z-auto  ">
        <div class="container mx-auto px-4 flex flex-row py-2">
            <UInput
                v-model="searchQuery"
                type="text"
                placeholder="Search..."
                class="w-full max-w-md mx-auto"
                @input="onSearch"
            />
            

            <ColorToggle />
            <UButton
                v-if="user"
                icon="i-lucide-user"
                variant="ghost"
                class="ml-2"
            />
            <UButton
                v-if="user"
                icon="i-lucide-log-out"
                variant="ghost"
                class="ml-2"
                @click="supabase.auth.signOut()"
            />
            <UButton
                v-if="!user"
                icon="i-lucide-log-in"
                variant="ghost"
                class="ml-2"
                :to="{ name: 'login' }"
            />
            <UButton
                v-if="!user"
                icon="i-lucide-user-plus"
                variant="ghost"
                class="ml-2"
                :to="{ name: 'register' }"
            />
            <UButton
                v-if="user"
                icon="i-lucide-settings"
                variant="ghost"
                class="ml-2"
                :to="{ name: 'settings' }"
            />
        </div>
            
    </nav>

</template>
<script lang="ts" setup>

const searchQuery = ref<string>("");
const emit = defineEmits<{
    (e: 'search', query: string): void;
}>();

const onSearch = () => {
    // Emit the search query to the parent component
    emit('search', searchQuery.value);
};
const supabase = useSupabaseClient();
const user = useSupabaseUser();
</script>

