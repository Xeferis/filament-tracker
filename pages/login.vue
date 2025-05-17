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
    <div class="flex flex-col h-fit w-64 p-4 bg-slate-500 rounded-sm">
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
      <UButton class="mt-5" @click="signIn">
        Sign In
      </UButton>
    </div>
  </div>
</template>
