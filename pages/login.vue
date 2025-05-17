<script setup lang="ts">
const supabase = useSupabaseClient()
const email = ref('')
const password = ref('')

const signIn = async () => {
  const {data, error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
    
  })
  if (error) console.log(error)
  if (data) redirect("/")
}
</script>
<template>
  <div class="flex w-full h-full justify-center content-center">
    <div class="flex flex-col w-64 p-4 bg-slate-500">
      <UInput
        placeholder="e-mail"
        v-model="email"
        type="email"
      />
      <UInput
        placeholder="password"
        v-model="password"
        type="password"
      />
      <UButton @click="signIn">
        Sign In
      </UButton>
    </div>
  </div>
</template>
