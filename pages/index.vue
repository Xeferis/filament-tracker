<script setup>
const supabase = useSupabaseClient()
const { data, status } = await supabase.from('filaments').select("id, type, amount, refill, manufacturer")
</script>
<template>
  <div class="flex flex-col justify-center items-center p-10 w-full">
    <h1 class="text-6xl text-center mb-14">Current Filament Supply</h1>
    <div class="flex flex-col w-4/5 md:w-full">
      <div class="flex justify-end items-center mb-2">
        <UButton to="/add">Add</UButton>
      </div>
      <UTable :data="data" class="flex-1" />
    </div>
  </div>
</template>