<script setup lang="ts">
const supabase = useSupabaseClient()
const email = ref('')
const password = ref('')

const signInWithOtp = async () => {
  const { error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  })
  if (error) console.log(error)
}
</script>
<template>
  <div class="flex flex-col w-full justify-center align-center">
    <UButton @click="signInWithOtp">
      Sign In with E-Mail + Password
    </UButton>
    <UInput
      v-model="email"
      type="email"
    />
    <UInput
      v-model="password"
      type="password"
    />
  </div>
</template>
