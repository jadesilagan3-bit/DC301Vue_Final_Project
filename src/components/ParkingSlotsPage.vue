<script setup>
defineProps({ slots: Array, freeCount: Number, notice: String })
defineEmits(['add', 'remove'])
</script>
<template>
  <div class="heading"><div><p class="eyebrow">PARKING AREA</p><h1>Parking Slots</h1><p class="muted">See which parking spaces are free or in use.</p></div><button class="primary" @click="$emit('add')">+ Add Slot</button></div>
  <p v-if="notice" class="notice">{{ notice }}</p>
  <section class="stats"><article class="stat"><span>Total Slots</span><strong>{{ slots.length }}</strong></article><article class="stat"><span>Available</span><strong>{{ freeCount }}</strong></article><article class="stat"><span>Occupied</span><strong>{{ slots.length - freeCount }}</strong></article></section>
  <section class="slot-grid"><article v-for="s in slots" :key="s.code" class="slot-card"><div class="slot-top"><strong>{{ s.code }}</strong><span :class="['status', s.occupied ? 'parked' : 'done']">{{ s.occupied ? 'Occupied' : 'Available' }}</span></div><p>{{ s.type }} parking</p><small>{{ s.occupied ? 'Vehicle is using this space.' : 'Ready for a new vehicle.' }}</small><button v-if="!s.occupied" class="text-button" @click="$emit('remove', s)">Remove slot</button></article></section>
</template>
