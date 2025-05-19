<script setup>
const config = useRuntimeConfig()
const supabase = useSupabaseClient()
const user = useSupabaseUser()

const { data: filaments } = await useAsyncData('filaments', () => {
  const { data, status } = supabase.from('filaments').select("id, type, amount, refill, manufacturer, location")

  return data
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
    <UTable :data="filaments" />
  </div>
</template>