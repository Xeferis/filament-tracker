<script setup>
const supabase = useSupabaseClient()
const { data, status } = await supabase.from('filaments').select("id, type, amount, refill, manufacturer, location")
</script>
<template>
  <div class="flex flex-col justify-center items-center w-full">
    <UTable :data="data" />
  </div>
</template>