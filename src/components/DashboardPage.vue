<script setup>
defineProps({ records: Array, parkedCount: Number, freeCount: Number, slots: Array, totalPaid: Number, money: Function })
defineEmits(['register', 'view-records'])
</script>
<template>
  <div class="heading"><div><p class="eyebrow">OVERVIEW</p><h1>Dashboard</h1><p class="muted">A quick look at your parking area.</p></div><button class="primary" @click="$emit('register')">+ Register Parking</button></div>
  <section class="stats"><article class="stat"><span>Vehicles Currently Parked</span><strong>{{ parkedCount }}</strong></article><article class="stat"><span>Available Parking Slots</span><strong>{{ freeCount }}</strong></article><article class="stat"><span>Total Parking Slots</span><strong>{{ slots.length }}</strong></article><article class="stat"><span>Payments Received</span><strong>{{ money(totalPaid) }}</strong></article></section>
  <section class="panel"><div class="panel-title"><div><h2>Recent Parking Records</h2><p>Latest vehicles registered in the system.</p></div><button class="secondary" @click="$emit('view-records')">View records</button></div>
    <div v-if="records.length" class="table-wrap"><table><thead><tr><th>Owner</th><th>Plate Number</th><th>Slot</th><th>Parking Status</th></tr></thead><tbody><tr v-for="r in [...records].reverse().slice(0,5)" :key="r.id"><td>{{ r.owner }}</td><td>{{ r.plate }}</td><td>{{ r.slot }}</td><td><span :class="['status', r.exit ? 'done' : 'parked']">{{ r.exit ? 'Checked Out' : 'Currently Parked' }}</span></td></tr></tbody></table></div>
    <div v-else class="empty"><strong>No records yet</strong><p>Choose Register Parking to add your first vehicle.</p><button class="primary" @click="$emit('register')">Register first vehicle</button></div>
  </section>
</template>
