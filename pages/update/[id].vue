<script setup lang="ts">
const { id } = useRoute().params

import type { RadioGroupItem, RadioGroupValue } from '@nuxt/ui'
import colorList from '~/composables/colorList'
import manufacturerList from '~/composables/manufacturerList'
import materialsList from '~/composables/materialsList'
import statusList from '~/composables/statusList'
const supabase = useSupabaseClient()

const { data: filaments_selected} = await useAsyncData('filaments_selected', async () => {
  const { data } = await supabase.from('filaments').select("id, type, refill, manufacturer, status, item_number, color, material, locations(*)").eq('id', id).limit(1)
  return data
})

const dd_loading = ref(false)
const type = ref(filaments_selected.value[0].type)
const item_number = ref(filaments_selected.value[0].item_number)
const refill = ref(filaments_selected.value[0].refill)
const manufacturer = ref(manufacturerList())
const manu_selected = ref(filaments_selected.value[0].manufacturer)
const color = ref(colorList())
const clr_selected = ref(filaments_selected.value[0].color)
const material = ref(materialsList())
const mtrl_selected = ref(filaments_selected.value[0].material)
const dd_selected = ref({
    label: filaments_selected.value[0].locations.description,
    value: filaments_selected.value[0].locations.id,
})
const dd_value = ref([])
const dd_status_sel = ref<RadioGroupValue>(filaments_selected.value[0].status)


const dd_status = ref<RadioGroupItem[]>(statusList())


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
  item_number: string
  type: string
  color: string
  material: string
  status: number
  refill: boolean
  manufacturer: string
  location_id: number
}

const errortoast = useToast()

const updateFilament = async () => {
    if (type.value && dd_status_sel.value && clr_selected.value && mtrl_selected.value && manu_selected.value && dd_selected.value.value) {
        console.log('update filament')
        console.log(type.value)
        console.log(item_number.value)
        console.log(dd_status_sel.value)
        console.log(clr_selected.value)
        console.log(mtrl_selected.value)
        console.log(refill.value)
        console.log(manu_selected.value)
        console.log(dd_selected.value.value)

        const { error } = await supabase
            .from('filaments')
            .update({
                type: type.value,
                item_number: item_number.value,
                status: dd_status_sel.value,
                color: clr_selected.value,
                material: mtrl_selected.value,
                refill: refill.value,
                manufacturer: manu_selected.value,
                location: dd_selected.value.value
            })
            .eq('id', id)

        if (error) {
            console.error('Error updating filament:', error)
            errortoast.add({
            title: 'Fehler',
            description: 'Fehler beim Aktualisieren des Filaments',
            color: 'error',
            })
        } else {
            errortoast.add({
            title: 'Erfolg',
            description: 'Filament erfolgreich aktualisiert',
            color: 'success',
            })
            navigateTo('/')
        }
    }
}

</script>
<template>
  <div class="flex flex-col justify-center items-center p-10 w-full">
    <h3 class="text-3xl md:text-4xl text-center mb-14">Add new Filament</h3>
    <div class="flex flex-col justify-center items-center md:w-4/5 w-full">
      <div class="flex justify-start items-center mb-4 md:w-1/3 w-full">
        <UButton to="/">Back</UButton>
      </div>
      <div class="flex flex-col h-fit w-80 p-4  bg-neutral-400 dark:bg-neutral-800 shadow-2xl dark:shadow-neutral-200 rounded-xl">
        <h2 class="text-2xl text-center mb-5">Updating Filament: #{{ id }}</h2>
        <h3 class="text-center mb-5">
          Please enter the filament details you want to update:
        </h3>

        <div class="my-2">
          <USelectMenu size="xl" class="w-full" v-model="manu_selected" placeholder="Hersteller" :items="manufacturer"/>
          <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>
        
        <div class="my-2">
        <USelectMenu size="xl" class="w-full"  v-model="clr_selected" placeholder="Farbe" :items="color"/>
        <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>

        <div class="my-2">
          <USelectMenu class="w-full" size="xl"  v-model="mtrl_selected" placeholder="Material" :items="material"/>
          <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>

        <div class="my-2">
          <UInput
            size="xl"
            class="w-full" 
            placeholder="Artikelnummer"
            v-model="item_number"
          />
        </div>

        <div class="my-2">
          <UInput
            size="xl"
            class="w-full" 
            placeholder="Bezeichnung"
            v-model="type"
          />
          <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>

        <div class="my-2 ">
          <URadioGroup class="w-full" variant="card" v-model="dd_status_sel" :items="dd_status"/>
          <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>

        <div class="my-2 flex justify-between items-center">
          <p>Nachfüllpackung: </p>
          <USwitch v-model="refill" size="xl"/>
        </div>
        
        <div class="my-2">
          <USelectMenu class="w-full" size="xl" :loading="dd_loading" v-model="dd_selected" placeholder="Select location" :items="dd_value"/>
          <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>
        <UButton size="xl" color="secondary" class="mt-5 justify-center" loading-auto @click="updateFilament">
          Update Filament
        </UButton>
      </div>
    </div>
  </div>
</template>