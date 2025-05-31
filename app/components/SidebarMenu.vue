<template>
    <UCard variant="solid" class="w-64 rounded-none fixed left-0 top-0 h-full ">
        <template #header >
            <h2 class="text-lg font-semibold">Menu</h2>
</template>
        <nav class="flex flex-col space-y-2 p-4">
            <UButton
                label="Home"
                icon="i-lucide-home"
                to="/"
                variant="ghost"
                class="justify-start"
            />
            <UButton
                label="Products"
                icon="i-lucide-box"
                to="/products"
                variant="ghost"
                class="justify-start"
            />
            <UButton
                label="Settings"
                icon="i-lucide-settings"
                to="/settings"
                variant="ghost"
                class="justify-start"
            />
        </nav>
           
    </UCard>
</template>
<script lang="ts" setup>




</script>