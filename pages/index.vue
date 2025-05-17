<script setup>
import { createClient } from '@supabase/supabase-js'
const config = useRuntimeConfig()
const supabase = createClient(config.public.supabaseUrl, config.public.supabaseAnonKey)
const filaments = ref([])
async function getfilaments() {
  const { data } = await supabase.from('filaments').select()
  filaments.value = data
}
onMounted(() => {
  getfilaments()
})

const LogOut = async () => {
  console.log("sign out user")
  const { error } = await supabase.auth.signOut()
  if (error) console.log(error)
}
</script>
<template>
  <UButton @click="logout">LogOut</UButton>
  <ul>
    <li v-for="filament in filaments" :key="filament.id">{{ filament.type }}</li>
  </ul>
</template>