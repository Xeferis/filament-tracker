<script setup lang="ts">
const supabase = useSupabaseClient()
const email = ref('')
const password = ref('')

const signIn = async () => {
  const {data, error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  })
  if (error) {
    console.log(error)
  } else {
    navigateTo("/")
  }
}
</script>
<template>
  <div class="flex w-full h-full justify-center items-center">
    <h1 class="text-3xl text-center mb-5">Filament Tracker</h1>
    <div class="flex flex-col h-fit w-80 p-4 bg-neutral-800 shadow-2xl shadow-neutral-200 rounded-xl">
      <h2 class="text-2xl text-center mb-5">Sign In</h2>
      <h3 class="text-center mb-5">
        Please enter your credentials to sign in
      </h3>
      <UInput
        class="my-2"
        placeholder="e-mail"
        v-model="email"
        type="email"
      />
      <UInput
        class="my-2"
        placeholder="password"
        v-model="password"
        type="password"
      />
      <UButton size="xl" class="mt-5 text-center" @click="signIn">
        Sign In
      </UButton>
    </div>
  </div>
</template>
