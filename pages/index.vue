<script setup>
const config = useRuntimeConfig()
const supabase = useSupabaseClient()
const user = useSupabaseUser()
const filaments = ref([])

async function getfilaments() {
  const { data, error } = await supabase.from('filaments').select("id, type, amount, refill, manufacturer, location")
  filaments.value = data
}
onMounted(() => {
  getfilaments()
})

const LogOut = async () => {
  console.log("sign out user")
  const { error } = await supabase.auth.signOut()
  if (error) {
    console.log(error)
  } else {
    navigateTo("/login")
  }
}

</script>
<template>
  <div>
    <p>logged in User:</p>
    <p>{{ user.email }}</p>
  </div>
  <UButton @click="LogOut">LogOut</UButton>
  <div class="flex justify-center items-center w-full">
    <table class="table-auto">
      <thead>
        <tr>
          <th>Type</th>
          <th>Hersteller</th>
          <th>refill</th>
          <th>Amount</th>
          <th>Location</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="fil in filaments">
          <td>{{fil.type}}</td>
          <td>{{fil.manufacturer}}</td>
          <td>{{fil.refill}}</td>
          <td>{{fil.amount}}</td>
          <td>{{fil.location}}</td>
        </tr>
      </tbody>
    </table>
    <UTable
      :data="filaments"
      :columns="[
        { key: 'type', label: 'Type' },
        { key: 'manufacturer', label: 'Hersteller' },
        { key: 'refill', label: 'refill' },
        { key: 'amount', label: 'Amount' },
        { key: 'location', label: 'Location' }
      ]"
      :options="{ perPage: 10 }"
      :pagination="{ page: 1, rowsPerPage: 10 }"
      :loading="false"
      :loadingText="'Loading...'"
      :noDataText="'No data available'"
      :noResultsText="'No results found'"
      :emptyText="'No data available'"
      :emptySlot="false"
      :showPagination="true"
      :showPageSize="true"
      :showPageSizes="[5, 10, 20]"
      :showPageSizeSelect="true"
      :showPageSizeSelectAll="true" />
  </div>
</template>