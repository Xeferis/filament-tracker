<script setup lang="ts">
import type { TableColumn, TableRow } from '@nuxt/ui'
import type { Column } from '@tanstack/vue-table'
import type { RealtimeChannel } from '@supabase/supabase-js'
import { h, resolveComponent } from 'vue'
import { UAvatar } from '#components'
let realtimeChannel: RealtimeChannel
const UBadge = resolveComponent('UBadge')
const UButton = resolveComponent('UButton')
const UDropdownMenu = resolveComponent('UDropdownMenu')
const UAvatar = resolveComponent('UAvatar')

const supabase = useSupabaseClient()
const open_modal = ref(false)
const open_modal_del = ref(false)

const { data: filaments, refresh: refreshFilaments } = await useAsyncData('filaments', async () => {
  const { data } = await supabase.from('filaments').select("id, type, refill, manufacturer, status, item_number, color, material, locations(description)")
  return data
})

// console.log(data) //#DEBUG

const modal_id = ref()
const modal_item_number = ref()
const modal_location = ref()
const modal_type = ref()
const modal_color = ref()
const modal_material = ref()
const modal_status = ref()
const modal_refill = ref()
const modal_manufacturer = ref()


type filament = {
  id: Number
  item_number: String
  location: String
  type: String
  color: String
  material: String
  status: Number
  refill: Boolean
  manufacturer: String
}

const columns: TableColumn<filament>[] = [
  {
    accessorKey: 'item_number',
    header: ({ column }) => getHeader(column, 'Artikelnummber'),
    cell: ({ row }) => h('div', { class: 'flex items-center gap-3' }, [
      h(UAvatar, { src: `filaments/${row.getValue('item_number')}.jpg`, alt: row.getValue('item_number'), size: 'md', class: 'mr-2' }),
      h('p', {}, row.getValue('item_number'))
    ])
  },
  {
    accessorKey: 'locations.description',
    header: ({ column }) => getHeader(column, 'Ort'),
  },
  {
    accessorKey: 'type',
    header: ({ column }) => getHeader(column, 'Bezeichnung'),
  },
  {
    accessorKey: 'color',
    header: ({ column }) => getHeader(column, 'Farbe'),
  },
  {
    accessorKey: 'material',
    header: ({ column }) => getHeader(column, 'Material'),
  },
  {
    accessorKey: 'status',
    header: ({ column }) => getHeader(column, 'Status'),
    cell: ({ row }) => {
      const color = {
        1: 'neutral' as const,
        2: 'warning' as const,
        3: 'secondary' as const
      }[row.getValue('status') as string]
      
      const text = {
        1: 'Bestellung geplant' as const,
        2: 'Bestellt' as const,
        3: 'Eingelagert' as const
      }[row.getValue('status') as string]


      return h(UBadge, { variant: 'subtle', color }, () =>
        text
      )
    }
  },
  {
    accessorKey: 'refill',
    header: ({ column }) => getHeader(column, 'Refill'),
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
    header: ({column}) => h('div', { class: 'text-right' }, getHeader(column, 'Hersteller')),
    cell: ({ row }) => {
      return h('div', { class: 'text-right' }, row.getValue('manufacturer'))
    }
  },
]
function getHeader(column: Column<filament>, label: string) {
  const isSorted = column.getIsSorted()

  return h(
    UDropdownMenu,
    {
      content: {
        align: 'start'
      },
      'aria-label': 'Actions dropdown',
      items: [
        {
          label: 'Asc',
          type: 'checkbox',
          icon: 'i-lucide-arrow-up-narrow-wide',
          checked: isSorted === 'asc',
          onSelect: () => {
            if (isSorted === 'asc') {
              column.clearSorting()
            } else {
              column.toggleSorting(false)
            }
          }
        },
        {
          label: 'Desc',
          icon: 'i-lucide-arrow-down-wide-narrow',
          type: 'checkbox',
          checked: isSorted === 'desc',
          onSelect: () => {
            if (isSorted === 'desc') {
              column.clearSorting()
            } else {
              column.toggleSorting(true)
            }
          }
        }
      ]
    },
    () =>
      h(UButton, {
        color: 'neutral',
        variant: 'ghost',
        label,
        icon: isSorted
          ? isSorted === 'asc'
            ? 'i-lucide-arrow-up-narrow-wide'
            : 'i-lucide-arrow-down-wide-narrow'
          : 'i-lucide-arrow-up-down',
        class: '-mx-2.5 data-[state=open]:bg-elevated',
        'aria-label': `Sort by ${isSorted === 'asc' ? 'descending' : 'ascending'}`
      })
  )
}

const sorting = ref([
  {
    id: 'item_number',
    desc: false
  }
])

const errortoast = useToast()

function switchModal() {
  open_modal.value = !open_modal.value
  open_modal_del.value = !open_modal_del.value
}

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
    open_modal_del.value = false
  }
}

const globalFilter = useCookie('globalFilter')

const rowSelection = ref<Record<string, boolean>>({})

function onSelect(row: TableRow<filament>, e?: Event) {
  /* If you decide to also select the column you can do this  */
  open_modal.value = !open_modal.value
  modal_id.value = row.original.id
  modal_item_number.value = row.original.item_number
  modal_location.value = row.original.locations.description
  modal_type.value = row.original.type
  modal_color.value = row.original.color
  modal_material.value = row.original.material
  modal_status.value = row.original.status
  modal_refill.value = row.original.refill
  modal_manufacturer.value = row.original.manufacturer
  //console.log(row.original.locations.description)
  //console.log(row)
  //console.log(e)
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
  <UModal :dismissible="false" v-model:open="open_modal_del" title="ACHTUNG!">
    <template #body>
      <div class="flex flex-col justify-center items-center text-center pb">
        <p class="text-lg">Sind Sie sicher das Sie das Filament löschen möchten?</p>
        <p class="text-lg text-error">Diese Aktion kann nicht rückgängig gemacht werden!</p>
      </div>
    </template>
    <template #footer>
      <UButton @click="deleteFilament(modal_id)" icon="i-lucide-trash" size="xl" color="error" variant="soft" loading-auto>Delete</UButton>
      <UButton @click="switchModal()" color="neutral" size="xl" variant="soft" loading-auto>Cancel</UButton>
    </template>
  </UModal>
  <UModal :dismissible="false" v-model:open="open_modal" title="Filament Details">
    <template #body>
      <div class="flex w-full my-2 mb-2 justify-center items-center">
        <NuxtImg
          :src="`filaments/${modal_item_number}.jpg`"
          sizes="200px md:400px"
          :alt="modal_item_number"
          class="rounded-lg"
          :custom="true"
          v-slot="{ src, isLoaded, imgAttrs }"
        >
          <!-- Show the actual image when loaded -->
          <img
            v-if="isLoaded"
            v-bind="imgAttrs"
            :src="src"
          >

          <!-- Show a placeholder while loading -->
          <img
            v-else
            class="md:w-[400px] w-[200px] rounded-lg"
            src="../public/placeholder.jpg"
            alt="placeholder"
          >
        </NuxtImg>
      </div>
      <div>
        <div class="flex justify-between items-center mb-2">
          <p>ID:</p>
          <p class="text-primary">{{ modal_id }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Artikelnummer:</p>
          <p class="text-primary">{{ modal_item_number }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Ort der Lagerung:</p>
          <p class="text-primary">{{ modal_location }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Bezeichnung:</p>
          <p class="text-primary">{{ modal_type }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Farbe:</p>
          <p class="text-primary">{{ modal_color }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Material:</p>
          <p class="text-primary">{{ modal_material }}</p>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Status:</p>
          <UBadge class="ml-2" variant="subtle" :color="modal_status === 1 ? 'neutral' : modal_status === 2 ? 'warning' : 'secondary'">
            {{ modal_status === 1 ? 'Bestellung geplant' : modal_status === 2 ? 'Bestellt' : 'Eingelagert' }}
          </UBadge>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Refill Roll:</p>
          <UBadge class="ml-2" variant="subtle" :color="modal_refill ? 'success' : 'error'">
            {{ modal_refill ? 'Ja' : 'Nein' }}
          </UBadge>
        </div>
        <div class="flex justify-between items-center mb-2">
          <p>Hersteller:</p>
          <p class="text-primary">{{ modal_manufacturer }}</p>
        </div>
        <div class="flex justify-between items-center mt-4">
          <UButton @click="switchModal()" icon="i-lucide-trash" color="error" loading-auto variant="soft">Delete</UButton>
          <UButton :to="'/update/' + modal_id" icon="i-lucide-book-up" color="info" variant="soft" loading-auto>Edit</UButton>
        </div>
      </div>
    </template>
  </UModal>
  <div class="flex flex-col justify-center items-center px-4 py-1 md:px-10 md:py-10 w-full">
    <h1 class="text-4xl md:text-6xl text-center mb-8 md:mb-14">Current Filament Supply</h1>
    <div class="flex flex-col md:w-4/5 w-full">
      <div class="flex justify-end items-center mb-2">
        <UButton size="xl" to="/add">Add</UButton>
      </div>
      <div class="flex px-4 py-3.5 border-b border-accented">
        <UInput v-model="globalFilter" class="w-full md:w-lg" size="xl" placeholder="Filter..." />
      </div>
      <UTable ref="table" v-model:sorting="sorting" v-model:global-filter="globalFilter" :data="filaments" :columns="columns" class="flex-1" v-model:row-selection="rowSelection" @select="onSelect" />
    </div>
  </div>
</template>