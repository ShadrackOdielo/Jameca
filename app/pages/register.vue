<template>
<div class="flex flex-col items-center justify-center gap-4 h-screen">
    
    <UCard class="w-96 p-6">
        <h1 class="font-bold text-2xl text-(--ui-primary) mb-4">Register</h1>
        <form @submit.prevent>
            <UInput placeholder="Username" type="text" required class="mb-4" />
            <UInput placeholder="Email" type="email" required class="mb-4" />
            <UInput placeholder="Password" type="password" required class="mb-4" />
            <UButton type="submit" label="Register" class="w-full" />
        </form>
        <p class="mt-4 text-sm text-gray-500">
            Already have an account? <a href="/login" class="text-(--ui-primary)">Login</a>
        </p>
    </UCard>
    <div class="mt-4 text-center text-sm text-gray-500">
        © 2023 Your Company. All rights reserved.
    </div>
    
  </div>
</template>
<script setup lang="ts">
</script>