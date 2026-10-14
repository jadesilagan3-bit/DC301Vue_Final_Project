
<script setup>
defineProps({
  form: Object,
  editingId: [Number, String],
  notice: String,
  availableSlots: Array,
  filteredRecords: Array,
  search: String,
  dateText: Function
})

defineEmits([
  'update:search',
  'save',
  'reset',
  'edit',
  'delete',
  'checkout'
])
</script>

<template>
  <div class="heading">
    <div>
      <p class="eyebrow">PARKING RECORDS</p>
      <h1>{{ editingId ? 'Edit Parking' : 'Register Parking' }}</h1>
      <p class="muted">Enter all details together in this one form.</p>
    </div>

    <button
      v-if="editingId"
      class="secondary"
      @click="$emit('reset')"
    >
      Cancel Edit
    </button>
  </div>

  <section class="panel form-panel">
    <form @submit.prevent="$emit('save')">
      <h2>Owner Information</h2>

      <div class="form-grid">
        <label>
          Owner Name *
          <input
            v-model="form.owner"
            required
            placeholder="e.g. Maria Santos"
          />
        </label>

        <label>
          Phone Number
          <input
            v-model="form.phone"
            type="tel"
            placeholder="e.g. 09171234567"
          />
        </label>
      </div>

      <h2>Vehicle Information</h2>

      <div class="form-grid three">
        <label>
          Plate Number *
          <input
            v-model="form.plate"
            required
            placeholder="e.g. ABC-1234"
          />
        </label>

        <label>
          Vehicle Type
          <select v-model="form.type">
            <option>Car</option>
            <option>SUV</option>
            <option>Motorcycle</option>
            <option>Van</option>
            <option>Truck</option>
          </select>
        </label>

        <label>
          Brand / Model
          <input
            v-model="form.brand"
            placeholder="e.g. Toyota Vios"
          />
        </label>
      </div>

      <h2>Parking Information</h2>

      <div class="form-grid three">
        <label>
          Parking Slot *
          <select v-model="form.slot" required>
            <option value="" disabled>Select slot</option>
            <option
              v-for="s in availableSlots"
              :key="s.code"
              :value="s.code"
            >
              {{ s.code }} ({{ s.type }})
            </option>
          </select>
        </label>

        <label>
          Entry Time *
          <input
            v-model="form.entry"
            type="datetime-local"
            required
          />
        </label>

        <label>
          Expected Exit Time
          <input
            v-model="form.exit"
            type="datetime-local"
          />
        </label>
      </div>

      <p class="muted">
        The expected exit time is your planned departure.
        It will not automatically check out the vehicle.
      </p>

      <h2>Payment Information</h2>

      <div class="form-grid three">
        <label>
          Amount (₱)
          <input
            v-model="form.amount"
            type="number"
            min="0"
            step="0.01"
            placeholder="0.00"
          />
        </label>

        <label>
          Payment Method
          <select v-model="form.method">
            <option>Cash</option>
            <option>GCash</option>
            <option>Card</option>
            <option>Bank Transfer</option>
          </select>
        </label>

        <label>
          Payment Status
          <select v-model="form.paid">
            <option>Unpaid</option>
            <option>Paid</option>
            <option>Pending</option>
          </select>
        </label>
      </div>

      <p v-if="notice" class="notice">{{ notice }}</p>

      <div class="form-actions">
        <span>* Required fields</span>

        <div>
          <button
            type="button"
            class="secondary"
            @click="$emit('reset')"
          >
            Clear Form
          </button>

          <button type="submit" class="primary">
            {{ editingId ? 'Update Record' : 'Save Record' }}
          </button>
        </div>
      </div>
    </form>
  </section>

  <section class="panel">
    <div class="panel-title">
      <div>
        <h2>Parking Records</h2>
        <p>Edit, delete, or check out a vehicle below.</p>
      </div>

      <input
        class="search"
        :value="search"
        @input="$emit('update:search', $event.target.value)"
        placeholder="Search owner or plate"
      />
    </div>

    <div v-if="filteredRecords.length" class="table-wrap">
      <table>
        <thead>
          <tr>
            <th>Owner / Vehicle</th>
            <th>Slot</th>
            <th>Entry</th>
            <th>Expected Exit</th>
            <th>Status</th>
            <th>Actions</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="r in filteredRecords" :key="r.id">
            <td>
              <strong>{{ r.owner }}</strong>
              <small>{{ r.plate }} · {{ r.type }}</small>
            </td>

            <td>{{ r.slot }}</td>
            <td>{{ dateText(r.entry) }}</td>
            <td>{{ dateText(r.exit) }}</td>

            <td>
              <span
                :class="[
                  'status',
                  r.status === 'Checked Out' ? 'done' : 'parked'
                ]"
              >
                {{ r.status || 'Currently Parked' }}
              </span>
            </td>

            <td>
              <div class="actions">
                <button
                  class="small"
                  @click="$emit('edit', r)"
                >
                  Edit
                </button>

                <button
                  v-if="r.status !== 'Checked Out'"
                  class="small"
                  @click="$emit('checkout', r)"
                >
                  Check Out
                </button>

                <button
                  class="small danger"
                  @click="$emit('delete', r)"
                >
                  Delete
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <p v-else class="empty-text">
      No parking records found.
    </p>
  </section>
</template>
