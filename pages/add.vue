<script setup lang="ts">

const supabase = useSupabaseClient()
const dd_loading = ref(false)
const type = ref('')
const status = ref()
const item_number = ref()
const refill = ref(false)
const manufacturer = ref([
  'BambuLab',
  '3DXTech',
  'PolyMaker',
  'Raise 3D',
  'Formfutura',
  'Flashforge',
  'Azurefilm',
  'eSun',
  'Extrudr',
  'TreeD',
  'PPprint',
  'Zaxe',
])
const manu_selected = ref()
const color = ref([
  'Schwarz',
  'Weiß',
  'Blau',
  'Grün',
  'Rot',
  'Grau',
  'Orange',
  'Natur',
  'Gelb',
  'Transparent',
  'Lila',
  'Silber',
  'Braun',
  'Pink',
  'Gold',
  'Türkis',
  'Beige',
  'Bronze',
  'Schwarz-Rot',
])
const clr_selected = ref()
const material = ref([
  'PLA',
  'PETG',
  'ABS',
  'Flexibel TPU/TPE',
  'Nylon (PA)',
  'Carbonfaser',
  'ASA',
  'PLA Max',
  'PLA Silk',
  'PVA/BVOH/BreakAway',
  'Glasfaser',
  'PVB',
  'Flexible',
  'PETG Max',
  'PC',
  'PP',
  'PC-ABS',
  'Wood',
  'PPS',
  'PC-FR',
])
const mtrl_selected = ref()
const dd_selected = ref()
const dd_value = ref([])
const dd_status_sel = ref([])


const dd_status = ref([
  {
    label: 'Bestellung geplant',
    value: 1,
  },
  {
    label: 'Bestellt',
    value: 2,
  },
  {
    label: 'Eingelagert',
    value: 3,
  },
])

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

const addFilament = async () => {
  if (type.value && dd_status_sel.value && clr_selected.value && mtrl_selected.value && manu_selected.value && dd_selected.value.value) {
    const { data, error } = await supabase
    .from('filaments')
    .insert<Filament>({
      type: type.value,
      item_number: item_number.value,
      status: dd_status_sel.value,
      color: clr_selected.value,
      material: mtrl_selected.value,
      refill: refill.value,
      manufacturer: manu_selected.value,
      location: dd_selected.value.value
    }).select()

    if (error) {
        console.error('Error inserting data:', error)
    } else {
        console.log('Data inserted successfully:', data)
        navigateTo('/')
    }
  } else {
    errortoast.add({
      title: 'Uh oh! Something went wrong.',
      description: 'Please enter all required values!',
    })
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
      <div class="flex flex-col h-fit w-80 p-4 bg-neutral-800 shadow-2xl shadow-neutral-200 rounded-xl">
        <h2 class="text-2xl text-center mb-5">Add new Filament</h2>
        <h3 class="text-center mb-5">
          Please enter the filament details:
        </h3>

        <div class="my-2">
          <UInputMenu size="xl" class="w-full" v-model="manu_selected" placeholder="Hersteller" :items="manufacturer"/>
          <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>
        
        <div class="my-2">
        <UInputMenu size="xl" class="w-full"  v-model="clr_selected" placeholder="Farbe" :items="color"/>
        <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>

        <div class="my-2">
          <UInputMenu class="w-full" size="xl"  v-model="mtrl_selected" placeholder="Material" :items="material"/>
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
          <URadioGroup class="w-full" size="xl" variant="card" v-model="dd_status_sel" default-value="1" :items="dd_status"/>
          <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>

        <div class="my-2 flex justify-between items-center">
          <p>Nachfüllpackung: </p>
          <USwitch v-model="refill" size="xl"/>
        </div>
        
        <div class="my-2">
          <UInputMenu class="w-full" size="xl" :loading="dd_loading" v-model="dd_selected" placeholder="Select location" :items="dd_value"/>
          <p class="text-red-600 text-xs mt-1 pl-2">required</p>
        </div>
        <UButton size="xl" class="mt-5 justify-center" @click="addFilament">
          Add Filament
        </UButton>
      </div>
    </div>
  </div>
</template>