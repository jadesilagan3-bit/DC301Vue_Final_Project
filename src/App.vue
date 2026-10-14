
<script setup>
import { computed, reactive, ref, watch } from 'vue'

import DashboardPage from './components/DashboardPage.vue'
import RegisterPage from './components/RegisterPage.vue'
import ParkingSlotsPage from './components/ParkingSlotsPage.vue'
import PaymentsPage from './components/PaymentsPage.vue'

const pages = [
  { id: 'dashboard', label: 'Dashboard' },
  { id: 'register', label: 'Register Parking' },
  { id: 'slots', label: 'Parking Slots' },
  { id: 'payments', label: 'Payments' }
]

const page = ref('dashboard')

const defaultSlots = [
  { code: 'A-01', type: 'Car', occupied: false },
  { code: 'A-02', type: 'Car', occupied: false },
  { code: 'A-03', type: 'Motorcycle', occupied: false },
  { code: 'B-01', type: 'Car', occupied: false },
  { code: 'B-02', type: 'Van', occupied: false },
  { code: 'B-03', type: 'Motorcycle', occupied: false }
]

function load(key, fallback) {
  try {
    const saved = localStorage.getItem('parkease-simple-' + key)
    return saved ? JSON.parse(saved) : fallback
  } catch {
    return fallback
  }
}

const records = ref(load('records', []))
const slots = ref(load('slots', defaultSlots))

// Keep the saved records and slots synchronized.
function syncSlots() {
  slots.value.forEach(slot => {
    slot.occupied = records.value.some(record =>
      record.slot === slot.code &&
      record.status !== 'Checked Out'
    )
  })
}

syncSlots()

watch(
  records,
  value => {
    localStorage.setItem(
      'parkease-simple-records',
      JSON.stringify(value)
    )
  },
  { deep: true }
)

watch(
  slots,
  value => {
    localStorage.setItem(
      'parkease-simple-slots',
      JSON.stringify(value)
    )
  },
  { deep: true }
)

const editingId = ref(null)
const search = ref('')
const notice = ref('')

function nowLocal() {
  return new Date(
    Date.now() - new Date().getTimezoneOffset() * 60000
  ).toISOString().slice(0, 16)
}

const form = reactive({
  owner: '',
  phone: '',
  plate: '',
  type: 'Car',
  brand: '',
  slot: '',
  entry: nowLocal(),
  exit: '',
  amount: '',
  method: 'Cash',
  paid: 'Unpaid'
})

const availableSlots = computed(() =>
  slots.value.filter(slot =>
    !slot.occupied ||
    records.value.some(record =>
      record.id === editingId.value &&
      record.slot === slot.code
    )
  )
)

const filteredRecords = computed(() => {
  const q = search.value.trim().toLowerCase()

  return [...records.value]
    .reverse()
    .filter(record =>
      !q ||
      [
        record.owner,
        record.plate,
        record.slot,
        record.brand
      ].some(value =>
        String(value || '').toLowerCase().includes(q)
      )
    )
})

const parkedCount = computed(() =>
  records.value.filter(
    record => record.status !== 'Checked Out'
  ).length
)

const totalPaid = computed(() =>
  records.value
    .filter(record => record.paid === 'Paid')
    .reduce(
      (sum, record) => sum + Number(record.amount || 0),
      0
    )
)

const freeCount = computed(() =>
  slots.value.filter(slot => !slot.occupied).length
)

function resetForm() {
  Object.assign(form, {
    owner: '',
    phone: '',
    plate: '',
    type: 'Car',
    brand: '',
    slot: '',
    entry: nowLocal(),
    exit: '',
    amount: '',
    method: 'Cash',
    paid: 'Unpaid'
  })

  editingId.value = null
  notice.value = ''
}

function saveRecord() {
  notice.value = ''

  if (
    !form.owner.trim() ||
    !form.plate.trim() ||
    !form.slot ||
    !form.entry
  ) {
    notice.value =
      'Please fill in owner name, plate number, parking slot, and entry time.'
    return
  }

  if (
    form.exit &&
    new Date(form.exit) < new Date(form.entry)
  ) {
    notice.value =
      'Expected exit time cannot be earlier than entry time.'
    return
  }

  const plate = form.plate.trim().toUpperCase()

  const duplicate = records.value.some(record =>
    record.plate.toUpperCase() === plate &&
    record.status !== 'Checked Out' &&
    record.id !== editingId.value
  )

  if (duplicate) {
    notice.value =
      'This vehicle is already parked. Check the plate number.'
    return
  }

  const old = records.value.find(
    record => record.id === editingId.value
  )

  const id = editingId.value ?? Date.now()

  const item = {
    id,
    owner: form.owner.trim(),
    phone: form.phone.trim(),
    plate,
    type: form.type,
    brand: form.brand.trim(),
    slot: form.slot,
    entry: form.entry,

    // This is the expected exit time, not the checkout action.
    exit: form.exit,

    // Only an actual checkout sets this status.
    status: old?.status === 'Checked Out'
      ? 'Checked Out'
      : 'Currently Parked',

    amount: Number(form.amount || 0),
    method: form.method,
    paid: form.paid
  }

  if (old) {
    records.value = records.value.map(record =>
      record.id === id ? item : record
    )
  } else {
    records.value.push(item)
  }

  syncSlots()

  notice.value = old
    ? 'Parking record updated.'
    : 'Parking record saved.'

  resetForm()
  page.value = 'register'
}

function editRecord(item) {
  editingId.value = item.id

  Object.assign(form, {
    owner: item.owner || '',
    phone: item.phone || '',
    plate: item.plate || '',
    type: item.type || 'Car',
    brand: item.brand || '',
    slot: item.slot || '',
    entry: item.entry || nowLocal(),
    exit: item.exit || '',
    amount: item.amount ?? '',
    method: item.method || 'Cash',
    paid: item.paid || 'Unpaid'
  })

  page.value = 'register'

  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
}

function deleteRecord(item) {
  if (
    !confirm(`Delete the parking record for ${item.plate}?`)
  ) {
    return
  }

  records.value = records.value.filter(
    record => record.id !== item.id
  )

  syncSlots()

  if (editingId.value === item.id) {
    resetForm()
  }

  notice.value = 'Parking record deleted.'
}

function checkOut(item) {
  if (item.status === 'Checked Out') {
    return
  }

  if (
    !confirm(`Mark ${item.plate} as checked out?`)
  ) {
    return
  }

  // Save the actual checkout time separately.
  item.actualExit = nowLocal()
  item.status = 'Checked Out'

  // Keep the original expected exit time in item.exit.
  syncSlots()
}

function addSlot() {
  const next = slots.value.length + 1
  const code = `C-${String(next).padStart(2, '0')}`

  if (slots.value.some(slot => slot.code === code)) {
    notice.value = 'That slot already exists.'
    return
  }

  slots.value.push({
    code,
    type: 'Car',
    occupied: false
  })

  notice.value = `Parking slot ${code} added.`
}

function removeSlot(slot) {
  if (slot.occupied) {
    notice.value = 'Cannot remove an occupied parking slot.'
    return
  }

  slots.value = slots.value.filter(
    item => item.code !== slot.code
  )

  notice.value = `Parking slot ${slot.code} removed.`
}

function money(amount) {
  return '₱' + Number(amount || 0).toLocaleString(
    'en-PH',
    {
      minimumFractionDigits: 2,
      maximumFractionDigits: 2
    }
  )
}

function dateText(value) {
  return value ? value.replace('T', ' ') : '—'
}

function goToRegister(clear = false) {
  page.value = 'register'

  if (clear) {
    resetForm()
  }
}
</script>

<template>
  <div class="app">
    <header class="header">
      <div class="brand">
        <span class="brand-icon">P</span>

        <div>
          <strong>ParkEase</strong>
          <small>Parking Management</small>
        </div>
      </div>

      <nav aria-label="Main navigation">
        <button
          v-for="item in pages"
          :key="item.id"
          :class="[
            'nav-button',
            { selected: page === item.id }
          ]"
          @click="page = item.id; notice = ''"
        >
          {{ item.label }}
        </button>
      </nav>
    </header>

    <main class="content">
      <DashboardPage
        v-if="page === 'dashboard'"
        :records="records"
        :parked-count="parkedCount"
        :free-count="freeCount"
        :slots="slots"
        :total-paid="totalPaid"
        :money="money"
        @register="goToRegister(true)"
        @view-records="goToRegister(false)"
      />

      <RegisterPage
        v-else-if="page === 'register'"
        :form="form"
        :editing-id="editingId"
        :notice="notice"
        :available-slots="availableSlots"
        :filtered-records="filteredRecords"
        :search="search"
        :date-text="dateText"
        @update:search="search = $event"
        @save="saveRecord"
        @reset="resetForm"
        @edit="editRecord"
        @delete="deleteRecord"
        @checkout="checkOut"
      />

      <ParkingSlotsPage
        v-else-if="page === 'slots'"
        :slots="slots"
        :free-count="freeCount"
        :notice="notice"
        @add="addSlot"
        @remove="removeSlot"
      />

      <PaymentsPage
        v-else
        :records="filteredRecords"
        :money="money"
        @edit="editRecord"
      />

      <footer>
        ParkEase · Simple Parking Management System
      </footer>
    </main>
  </div>
</template>
