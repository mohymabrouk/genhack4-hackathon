<template>
    <div class="section">
      <div class="card">
        <div class="input-group" style="margin-bottom: 0;">
          <label class="input-label">Select City</label>
          <select 
            class="input" 
            :value="selectedCity" 
            @change="$emit('select', $event.target.value)"
            :disabled="loading"
          >
            <option value="">-- Choose a city --</option>
            <option v-for="city in cities" :key="city" :value="city">
              {{ city }}
            </option>
          </select>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'CitySelector',
    props: {
      cities: Array,
      selectedCity: String,
      loading: Boolean
    },
    emits: ['select']
  }
  </script>
  