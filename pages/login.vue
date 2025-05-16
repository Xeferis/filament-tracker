<script setup lang="ts">
const supabase = useSupabaseClient()
const email = ref('')
const pw = ref('')

const signInWithOtp = async () => {
  const { error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: pw.value,
    options: {
      emailRedirectTo: 'http://localhost:3000/confirm',
    }
  })
  if (error) console.log(error)
}
</script>
<template>
  <div>
    <UButton @click="signInWithOtp">
      Sign In with E-Mail + Password
    </UButton>
    <UInput
      v-model="email"
      type="email"
    />
    <UInput
      v-model="pw"
      type="password"
    />
  </div>
</template>
