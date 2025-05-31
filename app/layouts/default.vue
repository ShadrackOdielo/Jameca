<template>
    <UCard :ui="{body:'min-h-screen',header:'w-full sticky top-0'}" class="w-screen min-h-screen relative rounded-none ring-0">
        <template #header>
            <NavbarMenu  />
        </template>
            <SidebarMenu />
    <div class="md:ml-64 flex-1 p-4 overflow-y-auto">

        <slot />
        </div>
        <template #footer>
            <div class="text-center text-sm text-gray-500">
                © 2023 Your Company. All rights reserved.
            </div>
            </template>
</UCard>
</template>