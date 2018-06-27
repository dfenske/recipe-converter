<template>
  <div class="ingredient-container">
    <div class="ingredient">
      <span>{{ format(amount) }} {{ unitAbbr }} {{ name }} <i v-if="showDelete" class="far fa-times-circle" @click="deleteItem"></i></span>
      <span v-if="showUnits">
        <select class="form-control fade-in inline" @change="changeUnits">
            <option :value="null" selected disabled>Change the units...</option>
            <option v-for="unit in relevantUnits" :key="unit.name" :value="unit.name">{{ unit.name }} ({{ unit.abbr }})</option>
        </select>
      </span>
    </div>
  </div>
</template>

<script>
import { volume, weight } from "../assets/conversions.js";

export default {
  name: "Ingredient",
  props: {
    name: String,
    abbr: String,
    amount: Number,
    unit: String,
    showDelete: Boolean,
    showUnits: Boolean
  },
  methods: {
    deleteItem() {
      this.$emit("delete-item", this.name);
    },
    format(value) {
      const wholeNumber =
        Math.floor(value) === 0 ? "" : Math.floor(value).toString();
      const decimal = value - wholeNumber;
      switch (decimal) {
        case 0.125:
          return wholeNumber + "1/8";
        case 0.2:
          return wholeNumber + "1/5";
        case 0.25:
          return wholeNumber + "¼";
        case 0.33:
          return wholeNumber + "1/3";
        case 0.375:
          return wholeNumber + "3/8";
        case 0.4:
          return wholeNumber + "2/5";
        case 0.5:
          return wholeNumber + "½";
        case 0.6:
          return wholeNumber + "3/5";
        case 0.625:
          return wholeNumber + "5/8";
        case 0.66:
          return wholeNumber + "2/3";
        case 0.75:
          return wholeNumber + "¾";
        case 0.8:
          return wholeNumber + "4/5";
        case 0.875:
          return wholeNumber + "7/8";
        default:
          return parseFloat(value.toFixed(2));
      }
    },
    changeUnits(e) {
      let newUnit = e.target.value || "";
      newUnit = newUnit.toLowerCase().replace(/\s+/g, '');

      const unitObj = volume[newUnit] || weight[newUnit];
      const multiplier = unitObj[this.unitProp]
      if (!multiplier) {
        this.errorMessage = 'You can\'t combine weights and volumes.';
      } else {
        const newAmount = this.amount * (1/multiplier);
        this.$emit("update-ingredient", this.name, newAmount, newUnit);
      }
    }
  },
  computed: {
    relevantUnits() {
      return weight[this.unitProp] ? weight : volume;
    },
    unitProp() {
      return this.unit.toLowerCase().replace(/\s+/g, '');
    },
    unitAbbr() {
      return weight[this.unitProp] ? weight[this.unitProp].abbr : volume[this.unitProp].abbr;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.ingredient-container {
  letter-spacing: 2px;
  padding-left: 15px;


  .ingredient {
    display: flex;
    min-height: 40px;
    
    span {
      width: 50%;
    }
    span select {
      opacity: 0;
    }

    &:hover {
      span select {
        opacity: 1;
      }
    }

    i {
      display: none;
    }

    &:hover i {
      display: inline-block;
      color: #dc3545;
      cursor: pointer;
      padding-left: 15px;

      &:hover {
        color: darkred;
      }
    }

    .fade-in {
      transition: opacity 0.4s ease-in-out;
    }
  }

  .inline {
    display: inline-block;
    width: auto;
    margin: 0;
    margin-left: 15px;
    margin-top: -15px;
  }
}
</style>
