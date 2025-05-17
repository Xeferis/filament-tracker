<script setup lang="ts">
const supabase = useSupabaseClient()
const email = ref('')
const password = ref('')

const signIn = async () => {
  const { error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  })
  if (error) console.log(error)
}
</script>
<template>
  <div class="flex w-full h-full justify-center content-center">
    <div class="flex flex-col w-64 p-4 bg-slate-500">
      <UInput
        v-model="email"
        type="email"
      />
      <UInput
        v-model="password"
        type="password"
      />
      <UButton @click="signIn">
        Sign In with E-Mail + Password
      </UButton>
    </div>
  </div>
</template>
