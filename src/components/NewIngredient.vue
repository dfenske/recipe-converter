<template>
  <div>
    <div class="form-inline">
      <label for="Amount">Amount</label>
      <input type="number" class="form-control margin-sm" v-model.number="amount" id="Amount">
    </div>
    <div class="form-inline">
      <label for="Unit">Units</label>
      <select class="form-control margin-sm" v-model="unit" id="Unit">
        <option v-for="unit in allUnits" :key="unit.name" :value="unit.name">{{unit.name ? `${unit.name} (${unit.abbr})` : 'No Unit'}}</option>
      </select>
    </div>
    <div class="form-inline">
      <label for="Name">Ingredient</label>
      <input class="form-control margin-sm" v-model="name" id="Name" placeholder="Eg. Flour" @keyup.enter="saveData">
    </div>
    <div class="add-button margin-top-sm margin-bottom-sm">
      <button type="button" class="btn blue" @click="saveData">ADD +</button>
    </div>
  </div>
</template>

<script>
import { volume, weight } from "../assets/conversions.js";

export default {
  data() {
    return {
      name: "",
      amount: null,
      unit: "No Unit"
    };
  },
  computed: {
    allUnits() {
      // return all units, and add a 'no unit' option.
      return {
        ...{ noUnit: { name: "", abbr: "" } },
        ...weight,
        ...volume
      };
    }
  },
  methods: {
    saveData() {
      if (this.amount > 0 && this.name) {
        this.$emit("add-ingredient", this.name, this.amount, this.unit);
      }
    }
  }
};
</script>

<style scoped lang='scss'>
@import "bootstrap/scss/bootstrap.scss";
.add-button {
  text-align: right;

  .btn {
    display: inline-block;
  }
}
@media screen and (max-width: map-get($grid-breakpoints, "md")) {
  .add-button {
    text-align: center;
  }
}
.form-inline {
  display: grid;
  grid-template-columns: 1fr 1fr;
}
.form-control {
  margin: 10px 0px;
}
</style>