<script setup>
const config = useRuntimeConfig()
const supabase = useSupabaseClient()
const user = useSupabaseUser()
const filaments = ref([])
const columns = [
  { key: 'id', label: 'ID' },
  { key: 'type', label: 'Bezeichnung' },
  { key: 'amount', label: 'Anzahl' },
  { key: 'refill', label: 'Refill' },
  // Add more keys as needed
]
async function getfilaments() {
  const { data, error } = await supabase.from('filaments').select()
  filaments.value = data
  console.log(data)
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
    <UTable :data="filaments" class="flex-1" />
  </div>
</template>