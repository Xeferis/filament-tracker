<script setup lang="ts">
import type { TableColumn } from '@nuxt/ui'
import { h, resolveComponent } from 'vue'
const UBadge = resolveComponent('UBadge')
const supabase = useSupabaseClient()
const { data, status } = await supabase.from('filaments').select("id, type, amount, refill, manufacturer")

type filament = {
  id: String
  type: String
  amount: Number
  refill: Boolean
  manufacturer: String
}

const columns: TableColumn<filament>[] = [
  {
    accessorKey: 'id',
    header: '#',
    cell: ({ row }) => `#${row.getValue('id')}`
  },
  {
    accessorKey: 'type',
    header: 'Bezeichnung',
  },
  {
    accessorKey: 'amount',
    header: 'Anzahl'
  },
  {
    accessorKey: 'refill',
    header: 'Refill Roll',
    cell: ({ row }) => {
      const color = {
        true: 'success' as const,
        false: 'error' as const
      }[row.getValue('refill') as string]

      return h(UBadge, { class: 'capitalize', variant: 'subtle', color }, () =>
        row.getValue('refill')
      )
    }
  },
  {
    accessorKey: 'manufacturer',
    header: 'Hersteller'
  },
]
</script>
<template>
  <div class="flex flex-col justify-center items-center p-10 w-full">
    <h1 class="text-6xl text-center mb-14">Current Filament Supply</h1>
    <div class="flex flex-col md:w-4/5 w-full">
      <div class="flex justify-end items-center mb-2">
        <UButton to="/add">Add</UButton>
      </div>
      <UTable :data="data" :columns="columns" class="flex-1" />
    </div>
  </div>
</template>