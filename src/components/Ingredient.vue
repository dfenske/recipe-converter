<template>
  <div>
    <div class="ingredient">
      {{ format(amount) }} {{ unit }} {{ name }} <i v-if="showDelete" class="far fa-times-circle" @click="deleteItem"></i>
      <span v-if="showUnits">
        <select>
          <option v-for="unit in allUnits" :key="unit.abbr" :value="unit.abbr">{{ unit.name }} ({{ unit.abbr }})</option>
        </select>
      </span>
    </div>
  </div>
</template>

<script>
import { volume, weight } from '../assets/conversions.js';

export default {
  name: 'Ingredient',
  props: {
    name: String,
    amount: Number,
    unit: String,
    showDelete: Boolean,
    showUnits: Boolean
  },
  methods: {
    deleteItem() {
      this.$emit('deleteItem', this.name);
    },
    format(value) {
      const wholeNumber = Math.floor(value) === 0 ? '' : Math.floor(value).toString();
      const decimal = value - wholeNumber;
      switch (decimal) {
        case 0.125:
          return wholeNumber + '1/8';
        case 0.2:
          return wholeNumber + '1/5';
        case 0.25:
          return wholeNumber + '¼';
        case 0.33: 
          return wholeNumber + '1/3';
        case 0.375:
          return wholeNumber + '3/8'
        case 0.4:
          return wholeNumber + '2/5';
        case 0.5: 
          return wholeNumber + '½';
        case 0.6:
          return wholeNumber + '3/5';
        case 0.625:
          return wholeNumber + '5/8'
        case 0.66:
          return wholeNumber + '2/3';
        case 0.75:
          return wholeNumber + '¾';
        case 0.8:
          return wholeNumber + '4/5';
        case 0.875:
          return wholeNumber + '7/8';
        default: 
          return value;
      }
    }
  },
  computed: {
    allUnits() {
      return {...weight, ...volume};
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>

.ingredient {
  display: inline-block;
}

span {
  letter-spacing: 2px;
  padding-left: 15px;
}

.ingredient:hover {
  i {
    display: inline-block;
    color: #dc3545;
    cursor: pointer;
    padding-left: 15px;

    &:hover {
      color: darkred;
    }
  }
}

.ingredient i {
  display: none;
}

.ingredient span{
  display: none;
}

.ingredient:hover span {
    display: inline-block;
}

</style>
