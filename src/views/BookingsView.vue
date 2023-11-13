<script setup>
import { ref, onMounted, watch } from 'vue'

const data = ref([])
const total = ref(0)
const loading = ref(false)
const sortField = ref('vote_count')
const sortOrder = ref('desc')
const defaultSortOrder = ref('desc')
const page = ref(1)
const perPage = ref(10)

const search = ref('')

watch(
  () => search.value,
  () => {
    loadAsyncData()
  }
)

const loadAsyncData = () => {
  const params = [
   
    `email=${search.value}`,
    `sort_by=${sortField.value}.${sortOrder.value}`,
    `page=${page.value}`,
  ].join('&')
  loading.value = true
  fetch(`/api/bookings?${params}`)
    .then((response) => response.json())
    .then((result) => {
     
      let currentTotal = result.total
     

      total.value = currentTotal
      data.value = result.bookings.map((item) => {
     
        return item
      })
     
      loading.value = false
    })
    .catch((error) => {
      data.value = []
      total.value = 0
      loading.value = false
      throw error
    })
}


const onPageChange = (p) => {
  page.value = p
  loadAsyncData()
}


const onSort = (field, order) => {
  sortField.value = field
  sortOrder.value = order
  loadAsyncData()
}


const type = (value) => {
  const number = parseFloat(value)
  if (number < 6) {
    return 'is-danger'
  } else if (number >= 6 && number < 8) {
    return 'is-warning'
  } else if (number >= 8) {
    return 'is-success'
  }
}

onMounted(() => {
  loadAsyncData()
})
</script>

<template>
  <section>
    <input class="form-control" v-model="search" placeholder="Search..." />

    <o-table
      :data="data"
      :loading="loading"
      paginated
      backend-pagination
      :total="total"
      :per-page="perPage"
      aria-next-label="Next page"
      aria-previous-label="Previous page"
      aria-page-label="Page"
      aria-current-label="Current page"
      backend-sorting
      :default-sort-direction="defaultSortOrder"
      :default-sort="[sortField, sortOrder]"
      @page-change="onPageChange"
      @sort="onSort"
    >
      <o-table-column
        v-slot="props"
        field="original_title"
        label="Email"
        sortable
      >
        <RouterLink :to="`/booking/${props.row._id}`">
          {{ props.row.email }}
        </RouterLink>
      </o-table-column>
      <o-table-column
        v-slot="props"
        field="numTickets"
        label="NumTickets"
        numeric
        sortable
      >
        <span class="tag" :class="type(props.row.vote_average)">
          {{ props.row.numTickets }}
        </span>
      </o-table-column>
      <o-table-column
        v-slot="props"
        field="vote_count"
        label="Superhero"
        numeric
        sortable
      >
        {{ props.row.superhero }}
      </o-table-column>
      <o-table-column
        v-slot="props"
        field="release_date"
        label="Last Modified Date"
        sortable
        centered
      >
        {{
          props.row.modifiedAt
            ? new Date(props.row.modifiedAt).toLocaleDateString()
            : 'unknown'
        }}
      </o-table-column>
      <o-table-column v-slot="props" label="Payment" width="500">
        {{ props.row.payment }}
      </o-table-column>
    </o-table>
  </section>
</template>
