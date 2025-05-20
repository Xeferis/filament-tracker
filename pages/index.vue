<script setup lang="ts">
import type { TableColumn, TableRow } from '@nuxt/ui'
import { h, resolveComponent } from 'vue'
const UBadge = resolveComponent('UBadge')
const supabase = useSupabaseClient()
const { data, status } = await supabase.from('filaments').select("id, type, amount, refill, manufacturer, color, material, locations(description)")
// console.log(data) //#DEBUG

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

const globalFilter = ref('')

const rowSelection = ref<Record<string, boolean>>({})

function onSelect(row: TableRow<filament>, e?: Event) {
  /* If you decide to also select the column you can do this  */
  console.log(row.getAllCells)
  console.log(e)
}
</script>
<template>
  <div class="flex flex-col justify-center items-center p-10 w-full">
    <h1 class="text-6xl text-center mb-14">Current Filament Supply</h1>
    <div class="flex flex-col md:w-4/5 w-full">
      <div class="flex justify-end items-center mb-2">
        <UButton to="/add">Add</UButton>
      </div>
      <div class="flex px-4 py-3.5 border-b border-accented">
        <UInput v-model="globalFilter" class="max-w-sm" placeholder="Filter..." />
      </div>
      <UTable ref="table" v-model:global-filter="globalFilter" :data="data" :columns="columns" class="flex-1" v-model:row-selection="rowSelection" @select="onSelect" />
    </div>
  </div>
</template>