<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent } from '@nuxt/ui'

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Must be at least 8 characters')
})

type Schema = z.output<typeof schema>
const supabase = useSupabaseClient()
const state = reactive<Partial<Schema>>({
  email: undefined,
  password: undefined
})

const toast = useToast()
async function onSubmit(event: FormSubmitEvent<Schema>) {
  event.preventDefault()
  const {  data,error } = await supabase.auth.signInWithPassword({
    email: state.email as string,
    password: state.password as string,
  })

  if (error) {
    toast.add({
      title: 'Login Failed',
      description: error.message,
        color: 'error'
    })
  } else {
    toast.add({
      title: 'Login Successful',
      description: data.user?.email ? `Welcome back, ${data.user.email}` : 'Login successful',
      color: 'success'
    })
    // redirect to home
    useRouter().push('/')
  }
}
</script>

<template>
    <div class="flex flex-col items-center justify-center gap-4 h-screen">
        <UCard class="w-96 p-6">
            <h1 class="font-bold text-2xl text-(--ui-primary) mb-4">Login</h1>
            <p class="text-sm text-gray-500 mb-4">Please enter your credentials to login.</p>
            <slot />
        
  <UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
    <UFormField label="Email" name="email">
      <UInput v-model="state.email" />
    </UFormField>

    <UFormField label="Password" name="password">
      <UInput v-model="state.password" type="password" />
    </UFormField>

    <UButton type="submit">
      Submit
    </UButton>
  </UForm>
  </UCard>
        <div class="mt-4 text-center text-sm text-gray-500">
            © 2023 Your Company. All rights reserved.
        </div>
    </div>
</template>

