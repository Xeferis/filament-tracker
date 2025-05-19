<script setup>
const supabase = useSupabaseClient()
const { data, status } = await supabase.from('filaments').select("id, type, amount, refill, manufacturer")
</script>
<template>
  <div class="flex flex-col justify-center items-center p-10 w-full">
    <h1 class="text-6xl text-center mb-16">Current Filament Supply</h1>
    <UTable :data="data" class="flex-1" />
  </div>
</template>