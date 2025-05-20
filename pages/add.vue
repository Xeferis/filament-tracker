<script setup lang="ts">

const supabase = useSupabaseClient()
const dd_loading = ref(false)
const type = ref('')
const amount = ref(0)
const refill = ref(false)
const manufacturer = ref('')
const dd_selected = ref()
const dd_value = ref([])

const { data: options, pending, error } = await useAsyncData('supabase-options', async () => {
    dd_loading.value = true
    const { data, error } = await supabase.from('locations').select()

    if (error) throw error
    console.log(data)

    const helper = data?.map((item) => {
        return {
            label: item.description,
            value: item.id,
        }
    })
    console.log(helper)
    dd_value.value = helper || ["No locations found"]
    dd_loading.value = false
    return data
})

interface Filament {
  type: string
  amount: number
  refill: boolean
  manufacturer: string
  location_id: number
}

const addFilament = async () => {
  const { data, error } = await supabase
    .from('filaments')
    .insert<Filament>({
      type: type.value,
      amount: amount.value,
      refill: refill.value,
      manufacturer: manufacturer.value,
      location_id: dd_selected.value.value
    }).select()


    console.log("add filament")
    console.log(type.value)
    console.log(amount.value)
    console.log(refill.value)
    console.log(manufacturer.value)
    console.log(dd_selected.value.value)
}

</script>
<template>
  <div class="flex flex-col justify-center items-center p-10 w-full">
    <h3 class="text-4xl text-center mb-14">Add new Filament</h3>
    <div class="flex flex-col justify-center items-center w-4/5 md:w-full">
      <div class="flex justify-start items-center mb-4 w-1/3">
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
        <UInputNumber v-model="amount" />
        <UInput
          class="my-2"
          placeholder="Manufacturer"
          v-model="manufacturer"
        />
        <UCheckbox class="my-2" v-model="refill" name="refill" label="Refill" size="xl" />
        <UInputMenu class="my-2" :loading="dd_loading" v-model="dd_selected" placeholder="Select location" :items="dd_value"/>
        <UButton size="xl" class="mt-5 justify-center" @click="addFilament">
          Add Filament
        </UButton>
      </div>
    </div>
  </div>
</template>