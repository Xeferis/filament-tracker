<script setup>
const config = useRuntimeConfig()
const supabase = useSupabaseClient()
const user = useSupabaseUser()
const filaments = ref([])
const testfil = ref([
  { type: "PLA", manufacturer: "Prusa", refill: false, amount: 1, location: "A1" },
  { type: "PLA", manufacturer: "Prusa", refill: false, amount: 1, location: "A2" },
  { type: "PLA", manufacturer: "Prusa", refill: false, amount: 1, location: "A3" },
])

async function getfilaments() {
  const { data, error } = await supabase.from('filaments').select("id, type, amount, refill, manufacturer, location")
  filaments.value = data
  console.log("filaments", filaments.value)
  console.log("filamentsfull", filaments)
  console.log("test", testfil.value)
  console.log("testfull", testfil)
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
  <div class="flex flex-col justify-center items-center w-full">
    <table class="border-collapse border border-gray-400 table-auto">
      <thead>
        <tr>
          <th>Type</th>
          <th>Hersteller</th>
          <th>refill</th>
          <th>Amount</th>
          <th>Location</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="fil in filaments">
          <td>{{fil.type}}</td>
          <td>{{fil.manufacturer}}</td>
          <td>{{fil.refill}}</td>
          <td>{{fil.amount}}</td>
          <td>{{fil.location}}</td>
        </tr>
      </tbody>
    </table>
    <UTable :data="testfil" />
  </div>
</template>