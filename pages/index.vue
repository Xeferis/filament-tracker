<script setup>
const config = useRuntimeConfig()
const supabase = useSupabaseClient()
const user = useSupabaseUser()
const filaments = ref([])
async function getfilaments() {
  const { data: filaments } = await useAsyncData('filaments', async () => {
    const { data } = await client.from('filaments').select()
    return data
  })
  filaments.value = data
  if (error) console.log(error)
  console.log("Database Data")
  console.log(filaments.value)
}
onMounted(() => {
  getfilaments()
})

const LogOut = async () => {
  console.log("sign out user")
  const { error } = await supabase.auth.signOut()
  if (error) {
    console.log(error)
  } else {
    navigateTo("/login")
  }
}
</script>
<template>
  <div>
    <p>logged in User:</p>
    <p>{{ user.email }}</p>
  </div>
  <UButton @click="LogOut">LogOut</UButton>
  <div class="w-full">
    <ul>
      <li v-for="filament in filaments" :key="filament.id">{{ filament.type }}</li>
    </ul>
  </div>
</template>