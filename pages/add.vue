<script setup>
const supabase = useSupabaseClient()
const type = ref('')
const amount = ref()
const refill = ref(false)
const manufacturer = ref('')
const dd_value = ref([])

const loc_data = await supabase.from('locations').select()

dd_value.value.push(loc_data.data)
console.log(loc_data.data)

const addFilament = async () => {
  const { data, error } = await supabase
    .from('filaments')
    .insert([
      { type: type.value, amount: amount.value, manufacturer: manufacturer.value, refill: refill.value, location: location.value }
    ])
  if (error) {
    console.log(error)
  } else {
    console.log(data)
    navigateTo('/')
  }
}

</script>
<template>
  <div class="flex flex-col justify-center items-center p-10 w-full">
    <h3 class="text-4xl text-center mb-14">Add new Filament</h3>
    <div class="flex flex-col w-4/5 md:w-full">
      <div class="flex justify-end items-center mb-2">
        <UButton to="/">Back</UButton>
      </div>
      <div class="flex flex-col h-fit w-80 p-4 bg-neutral-800 shadow-2xl shadow-neutral-200 rounded-xl">
        <h2 class="text-2xl text-center mb-5">Add new Filament</h2>
        <h3 class="text-center mb-5">
          Please enter the filament details:
        </h3>
        <UInput
          class="my-2"
          placeholder="Filament Type"
          v-model="type"
        />
        <UInput
          class="my-2"
          placeholder="Amount"
          v-model="amount"
        />
        <UInput
          class="my-2"
          placeholder="Manufacturer"
          v-model="manufacturer"
        />
        <UCheckbox class="my-2" v-model="refill" name="refill" label="Refill" />
        <UInputMenu class="my-2" v-model="loc_data.data[0]" :options="loc_data.data" option-attribute="description" value-attribute="id"/>
        <UButton size="xl" class="mt-5 justify-center" @click="addFilament">
          Add Filament
        </UButton>
      </div>
    </div>
  </div>
</template>