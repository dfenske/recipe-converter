<template>
  <div>
    <div class="form-inline row">
      <label class="col-sm" for="Amount">Amount</label>
      <input type="number" class="form-control col-sm" v-model.number="amount" id="Amount">
    </div>
    <div class="form-inline row">
      <label class="col-sm" for="Unit">Units</label>
      <select class="form-control col-sm" id="Unit" v-model="unit" >
        <option v-for="unit in allUnits" :key="unit.name" :value="unit.name">{{ unit.name }} ({{ unit.abbr }})</option>
      </select>
    </div>
    <div class="form-inline row">
      <label class="col-sm" for="Name">Ingredient</label>
      <input class="form-control col-sm" v-model="name" id="Name" placeholder="Eg. Flour" @keyup.enter="saveData">
    </div>
    <div class="align-right">
      <button type="button" class="btn btn-success inline" @click="saveData">ADD +</button>
    </div>
  </div>
</template>

<script>
  import { volume, weight } from '../assets/conversions.js';

  export default {
    data() {
      return {
        name: '',
        amount: null,
        unit: 'Cup (c)',
        volume: volume,
        weight: weight
      }
    },
    computed: {
      allUnits() {
        return {...this.weight, ...this.volume};
      }
    },
    methods: {
      saveData() {
        if (this.amount > 0) {
          this.$emit('add-ingredient', this.name, this.amount, this.unit);
        }
      }
    }
  }
</script>

<style scoped lang='scss'>
.btn.inline {
  display: inline-block;
}
</style>