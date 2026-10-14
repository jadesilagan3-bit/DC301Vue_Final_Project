<script setup>
defineProps({ records: Array, money: Function })
defineEmits(['edit'])
</script>
<template>
  <div class="heading"><div><p class="eyebrow">PAYMENT RECORDS</p><h1>Payments</h1><p class="muted">Review the payment status for each parking record.</p></div></div>
  <section class="stats"><article class="stat"><span>Total Amount Recorded</span><strong>{{ money(records.reduce((sum, r) => sum + Number(r.amount || 0), 0)) }}</strong></article><article class="stat"><span>Paid</span><strong>{{ money(records.filter(r => r.paid === 'Paid').reduce((sum, r) => sum + Number(r.amount || 0), 0)) }}</strong></article><article class="stat"><span>Unpaid / Pending</span><strong>{{ records.filter(r => r.paid !== 'Paid').length }}</strong></article></section>
  <section class="panel"><div class="panel-title"><div><h2>Payment List</h2><p>To change a payment, edit its parking record.</p></div></div>
    <div v-if="records.length" class="table-wrap"><table><thead><tr><th>Owner</th><th>Plate Number</th><th>Amount</th><th>Method</th><th>Payment Status</th><th></th></tr></thead><tbody><tr v-for="r in records" :key="r.id"><td>{{ r.owner }}</td><td>{{ r.plate }}</td><td>{{ money(r.amount) }}</td><td>{{ r.method }}</td><td><span :class="['status', r.paid === 'Paid' ? 'done' : 'waiting']">{{ r.paid }}</span></td><td><button class="small" @click="$emit('edit', r)">Edit payment</button></td></tr></tbody></table></div><p v-else class="empty-text">No payments yet. Add a record on Register Parking.</p>
  </section>
</template>
