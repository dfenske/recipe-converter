<template>
  <div class="ingredient padding-left-sm padding-bottom-sm">
      <span>
        {{ formatAsFraction(amount) }} {{ unitAbbr }} {{ name }} 
      </span>
      <span v-if="!isConvertedRecipe" class="text-right">
        <i class="far fa-times-circle" @click="deleteItem"></i>
      </span>
      <span v-if="isConvertedRecipe">
        <select v-if="unit" class="form-control margin-sm" @change="changeUnits">
            <option :value="null" selected disabled>Changeunits...</option>
            <option v-for="unit in relevantUnits" :key="unit.name" :value="unit.name">{{ unit.name }} ({{ unit.abbr }})</option>
        </select>
      </span>
    </div>
</template>

<script>
import { volume, weight } from "../assets/conversions.js";

export default {
  name: "Ingredient",
  props: {
    name: String,
    amount: Number,
    unit: String,
    isConvertedRecipe: Boolean
  },
  methods: {
    deleteItem() {
      this.$emit("delete-item", this.name);
    },
    formatAsFraction(value) {
      const wholeNumber =
        Math.floor(value) === 0 ? "" : Math.floor(value).toString();
      const decimal = value - wholeNumber;
      let prefix = wholeNumber ? wholeNumber + " " : "";
      switch (decimal) {
        case 0.125:
          return prefix + "1/8";
        case 0.2:
          return prefix + "1/5";
        case 0.25:
          return prefix + "¼";
        case 0.33:
          return prefix + "1/3";
        case 0.375:
          return prefix + "3/8";
        case 0.4:
          return prefix + "2/5";
        case 0.5:
          return prefix + "½";
        case 0.6:
          return prefix + "3/5";
        case 0.625:
          return prefix + "5/8";
        case 0.66:
          return prefix + "2/3";
        case 0.75:
          return prefix + "¾";
        case 0.8:
          return prefix + "4/5";
        case 0.875:
          return prefix + "7/8";
        default:
          return parseFloat(value.toFixed(2));
      }
    },
    changeUnits(e) {
      let newUnit = e.target.value;
      if (newUnit) {
        newUnit = newUnit.toLowerCase().replace(/\s+/g, "");
        this.$emit("update-ingredient", this.name, newUnit);
      }
    }
  },
  computed: {
    relevantUnits() {
      // only return the volume units if the current unit is a volume, and same for weight.
      if (this.unit) {
        return weight[this.unitProp] ? weight : volume;
      }
      // this handles the case where we have a unit-less ingredient (like eggs)
      return [];
    },
    unitAbbr() {
      if (this.unit) {
        return weight[this.unitProp]
          ? weight[this.unitProp].abbr
          : volume[this.unitProp].abbr;
      }
      return "";
    },
    unitProp() {
      // get the property name for this unit (no spaces, all lower case)
      return this.unit.toLowerCase().replace(/\s+/g, "");
    }
  }
};
</script>

<style lang="scss" scoped>
@import "bootstrap/scss/bootstrap.scss";

.ingredient {
  letter-spacing: 2px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-gap: 1em;

  // hide x and select by default
  span {
    select {
      opacity: 0;
      transition: opacity 1s ease-in-out;
    }
  }

  i {
    opacity: 0;
    transition: opacity 1s ease-in-out;
    padding-left: 15px;
    color: $red;
    cursor: pointer;
  }

  // if you hover over ingredient, show x and select
  &:hover {
    span select {
      opacity: 1;
      transition: opacity 0.4s ease-in-out;
    }

    i {
      opacity: 1;
      transition: opacity 0.4s ease-in-out;

      &:hover {
        color: darkred;
      }
    }
  }
}

// Mobile: show the hover items by default
@media screen and (max-width: map-get($grid-breakpoints, "md")) {
  .ingredient {
    i {
      opacity: 1;
    }
    span select {
      opacity: 1;
    }
  }
}
</style>
