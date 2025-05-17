<script setup>
const config = useRuntimeConfig()
const supabase = useSupabaseClient()
const user = useSupabaseUser()
const filaments = ref([])
const test_fil = ref([
  {
  amount: 1,
  created_at: "2025-05-17T07:14:37.469169+00:00",
  id: 2,
  location: "70b0528a-7108-4961-93b2-d6a72d047560",
  manufacturer: "Bambulab",
  refill: false,
  type: "Test Matte",
  },
  {
  amount: 1,
  created_at: "2025-05-17T07:14:37.469169+00:00",
  id: 3,
  location: "70b0528a-7108-4961-93b2-d6a72d047560",
  manufacturer: "Bambulab",
  refill: false,
  type: "Test glns",
  }
])
console.log("test data")
console.log(test_fil)
async function getfilaments() {
  const { data, error } = await supabase.from('filaments').select()
  filaments.value = data
  console.log(data)
  console.log("Database Data")
  console.log(filaments)
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
    <p v-for="fil in filaments">{{ fil }}</p>
    <UDivider></UDivider>
    <p v-for="fil2 in test_fil">{{ fil2 }}</p>

    <UTable :data="test_fil" class="flex-1" />
  </div>
</template>