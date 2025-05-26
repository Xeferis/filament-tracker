<script setup lang="ts">
import ChangeColor from '~/components/ChangeColor.vue'

const supabase = useSupabaseClient()
const user = useSupabaseUser()

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
    <div class="flex w-full h-24 mb-4 justify-evenly items-center">
      <div>
        <p>logged in as:</p>
        <p class="text-primary">{{ user?.email }}</p>
      </div>
      <div class="flex gap-2 md:gap-4">
        <ChangeColor />
        <UButton loading-auto @click="LogOut">LogOut</UButton>
      </div>
    </div>
    <slot />
  </div>
</template>