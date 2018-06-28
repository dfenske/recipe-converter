<template>
  <div class="ingredient">
      <span>
        {{ format(amount) }} {{ unitAbbr }} {{ name }} <i v-if="!converted" class="far fa-times-circle" @click="deleteItem"></i>
      </span>
      <span  v-if="converted">
        <select v-if="unit" class="form-control margin-sm" @change="changeUnits">
            <option :value="null" selected disabled>Change the units...</option>
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
    abbr: String,
    amount: Number,
    unit: String,
    converted: Boolean
  },
  methods: {
    deleteItem() {
      this.$emit("delete-item", this.name);
    },
    format(value) {
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
      if (this.unit) {
        return weight[this.unitProp] ? weight : volume;
      }
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
      return this.unit.toLowerCase().replace(/\s+/g, "");
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
@import "bootstrap/scss/bootstrap.scss";
.ingredient {
  letter-spacing: 2px;
  padding-left: 15px;
  display: flex;
  flex-wrap: wrap;
  min-height: 40px;
  min-width: 430px;

  span {
    flex: 8;
    min-width: 200px;
    select {
      opacity: 0;
      transition: opacity 1s ease-in-out;
    }
  }

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

  i {
    opacity: 0;
    flex: 1;
    transition: opacity 1s ease-in-out;
    padding-left: 15px;
    color: $red;
    cursor: pointer;
    float: right;
  }

  .form-control {
    display: inline-block;
    width: auto;
    margin: 0;
    margin-left: 15px;
    margin-top: -15px;
  }
}

@media screen and (max-width: map-get($grid-breakpoints, "md")) {
  .recipe ul {
    margin-left: -15px;
  }
  .ingredient i {
    opacity: 1;
  }
  .ingredient span select {
    opacity: 1;
  }
}
</style>
