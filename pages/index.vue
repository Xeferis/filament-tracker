<script setup lang="ts">
import type { TableColumn, TableRow } from '@nuxt/ui'
import type { RealtimeChannel } from '@supabase/supabase-js'
import { h, resolveComponent } from 'vue'
let realtimeChannel: RealtimeChannel
const UBadge = resolveComponent('UBadge')
const supabase = useSupabaseClient()
const open_modal = ref(false)

const { data: filaments, refresh: refreshFilaments } = await useAsyncData('filaments', async () => {
  const { data } = await supabase.from('filaments').select("id, type, amount, refill, manufacturer, color, material, locations(description)")
  return data
})

// console.log(data) //#DEBUG

const modal_id = ref()
const modal_location = ref()
const modal_type = ref()
const modal_color = ref()
const modal_material = ref()
const modal_amount = ref()
const modal_refill = ref()
const modal_manufacturer = ref()


type filament = {
  id: String
  location: String
  type: String
  color: String
  material: String
  amount: Number
  refill: Boolean
  manufacturer: String
}

const columns: TableColumn<filament>[] = [
  {
    accessorKey: 'id',
    header: '#ID',
    cell: ({ row }) => `#${row.getValue('id')}`
  },
  {
    accessorKey: 'locations.description',
    header: 'Ort der Lagerung',
  },
  {
    accessorKey: 'type',
    header: 'Bezeichnung',
  },
  {
    accessorKey: 'color',
    header: 'Farbe',
  },
  {
    accessorKey: 'material',
    header: 'Material',
  },
  {
    accessorKey: 'amount',
    header: () => h('div', { class: 'text-center' }, 'Anzahl'),
    cell: ({ row }) => {
      return h('div', { class: 'text-center font-bold' }, row.getValue('amount'))
    }
  },
  {
    accessorKey: 'refill',
    header: 'Refill Roll',
    cell: ({ row }) => {
      const color = {
        true: 'success' as const,
        false: 'error' as const
      }[row.getValue('refill') as string]

      const b2t = {
        true: 'ja' as const,
        false: 'nein' as const
      }[row.getValue('refill') as string]

      return h(UBadge, { class: 'capitalize', variant: 'subtle', color }, () =>
        b2t
      )
    }
  },
  {
    accessorKey: 'manufacturer',
    header: () => h('div', { class: 'text-right' }, 'Hersteller'),
    cell: ({ row }) => {
      return h('div', { class: 'text-right' }, row.getValue('manufacturer'))
    }
  },
]
const errortoast = useToast()

async function deleteFilament(id: string) {
  const { error } = await supabase.from('filaments').delete().eq('id', id)
  if (error) {
    console.error('Error deleting filament:', error)
    errortoast.add({
      title: 'Fehler',
      description: 'Fehler beim Löschen des Filaments',
      color: 'error',
    })
  } else {
    errortoast.add({
      title: 'Erfolg',
      description: 'Filament erfolgreich gelöscht',
      color: 'success',
    })
    // Refresh the data after deletion
    refreshNuxtData('filaments')
    open_modal.value = false
  }
}

const globalFilter = ref('')

const rowSelection = ref<Record<string, boolean>>({})

function onSelect(row: TableRow<filament>, e?: Event) {
  /* If you decide to also select the column you can do this  */
  open_modal.value = !open_modal.value
  modal_id.value = row.original.id
  modal_location.value = row.original.location
  modal_type.value = row.original.type
  modal_color.value = row.original.color
  modal_material.value = row.original.material
  modal_amount.value = row.original.amount
  modal_refill.value = row.original.refill
  modal_manufacturer.value = row.original.manufacturer
  console.log(row.original.id)
  console.log(e)
}

onMounted(() => {
  // Real time listener for new workouts
  realtimeChannel = supabase.channel('public:filaments').on(
    'postgres_changes',
    { event: '*', schema: 'public', table: 'filaments' },
    () => refreshFilaments()
  )

  realtimeChannel.subscribe()
})

  // Don't forget to unsubscribe when user left the page
onUnmounted(() => {
  supabase.removeChannel(realtimeChannel)
})

</script>
<template>
  <UModal :dismissible="false" v-model:open="open_modal" title="Filament Details">
    <template #body>
      <div>
        <div class="flex justify-between items-center mb-2">
          <p>ID:</p>
          <p>{{ modal_id }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Ort der Lagerung:</p>
          <p>{{ modal_location }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Bezeichnung:</p>
          <p>{{ modal_type }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Farbe:</p>
          <p>{{ modal_color }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Material:</p>
          <p>{{ modal_material }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Anzahl:</p>
          <div class="flex justify-center items-center">
            <UButton
              :disable="modal_amount > 0"
              class="mr-2"
              @click="modal_amount--"
              icon="i-lucide-minus"
              color="success"
              variant="soft"
            ></UButton>
            <p class="font-bold w-8 text-center">{{ modal_amount }}</p>
            <UButton
              :disable="modal_amount < 100"
              class="ml-2"
              @click="modal_amount++"
              icon="i-lucide-plus"
              color="success"
              variant="soft"
            ></UButton>
          </div>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Refill Roll:</p>
          <p>{{ modal_refill }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Hersteller:</p>
          <p>{{ modal_manufacturer }}</p>
        </div>
        <div class="flex justify-between items-center mt-4">
          <UButton @click="deleteFilament(modal_id)" icon="i-lucide-trash" color="error" variant="soft">Delete</UButton>
        </div>
      </div>
    </template>
  </UModal>
  <div class="flex flex-col justify-center items-center p-10 w-full">
    <h1 class="text-6xl text-center mb-14">Current Filament Supply</h1>
    <div class="flex flex-col md:w-4/5 w-full">
      <div class="flex justify-end items-center mb-2">
        <UButton to="/add">Add</UButton>
      </div>
      <div class="flex px-4 py-3.5 border-b border-accented">
        <UInput v-model="globalFilter" class="max-w-sm" placeholder="Filter..." />
      </div>
      <UTable ref="table" v-model:global-filter="globalFilter" :data="filaments" :columns="columns" class="flex-1" v-model:row-selection="rowSelection" @select="onSelect" />
    </div>
  </div>
</template>