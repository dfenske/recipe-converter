<template>
  <div id="app">
    <nav class="navbar blue">
      <h1>Convert My Recipe</h1>
    </nav>
    <div class="container margin-bottom-lg">
      <div class="user-directions text-bold text-center margin-bottom-lg">Enter your recipe's title, ingredients, and instructions and then enter the multiplier.</div>
      <h2 class="recipe-title no-print">
        <input class="form-control margin-sm" v-model.lazy="recipeTitle" placeholder="What is this recipe called?" />
      </h2>
      <div class="no-print original">
        <div class="recipe">
          <div v-if="recipe.length === 0" class="subtle no-print">Your recipe will show here. Enter ingredients below to start.</div>
          <div v-if="recipe.length > 0" class="subtle no-print hover-text margin-bottom-sm">Hover over an ingredient to delete it.</div>
          <ul>
            <li v-for="ingredient in recipe" :key="ingredient.name">
              <ingredient :name="ingredient.name" :amount="ingredient.amount" :unit="ingredient.unit" :converted="false" v-on:delete-item="deleteItem" ></Ingredient>
            </li>
          </ul>
        </div>
      </div>
      <div class="add-ingredients">
        <new-ingredient v-on:add-ingredient="addIngredient" />
      </div>
      <div class="instructions-input">
        <textarea class="form-control margin-sm no-print" placeholder="Enter the instructions here..." v-model.lazy="instructions" />
      </div>
      <div class="multiplier margin-top-sm no-print">
        <label for="Multiplier">x </label>
        <input class="form-control margin-sm" v-model.number="multiplier" id="Multiplier" type="number"/>
      </div>
      
      <hr class="no-print"/>
      <h2 class="new-recipe-title">{{ recipeTitle ? recipeTitle + ' x ' + multiplier : ''}} </h2>
      <div class="new recipe">
        <div v-if="recipe.length > 0" class="subtle hover-text no-print">Hover over an ingredient to change the units.</div>
        <div class="text-right"><i class="fas fa-print fa-lg no-print" @click="print"></i></div>
        <ul>
          <li v-for="ingredient in convertedRecipe" :key="ingredient.name">
            <ingredient :name="ingredient.name" :amount="ingredient.amount" :unit="ingredient.unit" :converted="true" v-on:update-ingredient="updateIngredient"></Ingredient>
          </li>
        </ul>
        <div class="instructions">{{ instructions }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import localforage from "localforage";
import Ingredient from "./components/Ingredient.vue";
import NewIngredient from "./components/NewIngredient.vue";
import { volume, weight } from "./assets/conversions.js";

export default {
  name: "app",
  components: {
    Ingredient,
    NewIngredient
  },
  data() {
    return {
      recipeTitle: "",
      multiplier: 1,
      recipe: [],
      units: {
        volume,
        weight
      },
      instructions: ""
    };
  },
  async mounted() {
    localforage.getItem("recipe", (err, value) => this.mountRecipe(err, value));
    localforage.getItem("recipeTitle", (err, value) =>
      this.mountTitle(err, value)
    );
    localforage.getItem("multiplier", (err, value) =>
      this.mountMultiplier(err, value)
    );
    localforage.getItem("instructions", (err, value) =>
      this.mountInstructions(err, value)
    );
  },
  methods: {
    mountRecipe(err, value) {
      if (err || !value) {
        localforage.setItem("recipe", []);
        this.recipe = [];
      } else {
        this.recipe = value;
      }
    },
    mountTitle(err, value) {
      if (err || !value) {
        localforage.setItem("recipeTitle", "");
        this.recipeTitle = "";
      } else {
        this.recipeTitle = value;
      }
    },
    mountMultiplier(err, value) {
      if (err || !value) {
        localforage.setItem("multiplier", 1);
        this.multiplier = 1;
      } else {
        this.multiplier = value;
      }
    },
    mountInstructions(err, value) {
      if (err || !value) {
        localforage.setItem("instructions", "");
        this.instructions = "";
      } else {
        this.instructions = value;
      }
    },
    addIngredient(name, amount, unit) {
      this.recipe.push({
        name: name,
        amount: amount,
        unit: unit
      });
    },
    deleteItem(name) {
      this.recipe.splice(this.recipe.findIndex(x => x.name === name), 1);
    },
    updateIngredient(name, newUnit) {
      // find this ingredient in the recipe
      const index = this.recipe.findIndex(x => x.name === name);
      const currIngredient = this.recipe[index];
      const currAmount = currIngredient.amount;
      const currUnit = currIngredient.unit.toLowerCase().replace(/\s+/g, "");

      // find the new unit object in the conversions
      const newUnitObj = volume[newUnit] || weight[newUnit];
      const unitMultiplier = newUnitObj[currUnit];

      if (unitMultiplier) {
        const newAmount = currAmount * (1 / unitMultiplier);
        this.recipe.splice(index, 1, {
          name: name,
          amount: newAmount,
          unit: newUnit
        });
      }
    },
    print() {
      window.print();
    }
  },
  watch: {
    recipe: function() {
      localforage.setItem("recipe", this.recipe);
    },
    multiplier: function() {
      localforage.setItem("multiplier", this.multiplier);
    },
    recipeTitle: function() {
      localforage.setItem("recipeTitle", this.recipeTitle);
    },
    instructions: function() {
      localforage.setItem("instructions", this.instructions);
    }
  },
  computed: {
    convertedRecipe() {
      return this.recipe.map(ingredient => {
        return {
          name: ingredient.name,
          amount: ingredient.amount * this.multiplier,
          unit: ingredient.unit
        };
      });
    }
  }
};
</script>

<style scoped lang='scss'>
@import "bootstrap/scss/bootstrap.scss";
:root {
  --grid-template-num-columns: 1fr 1fr;
  --first-col: 1 / 2;
  --second-col: 2 / 3;
}

.app * {
  box-sizing: border-box;
}

.container {
  --grid-template-num-columns: 1fr 1fr;
  display: grid;
  grid-template-columns: var(--grid-template-num-columns);
  grid-gap: 1em;
}

.recipe-title input {
  width: 100%;
}

.original {
  grid-column: var(--first-col);
}

.add-ingredients {
  grid-column: var(--second-col);
}

.user-directions,
.recipe-title,
.instructions-input,
.multiplier,
.new-recipe-title,
.recipe.new {
  grid-column: 1 / -1;
}

@media screen and (max-width: map-get($grid-breakpoints, "md")) {
  .container {
    --grid-template-num-columns: 1fr;
    --first-col: 1 / -1;
    --second-col: 1 / -1;
  }

  .recipe ul {
    margin-left: -15px;
  }
  i.fa-print {
    display: none;
  }
  .subtle.hover-text {
    display: none;
  }
}
@media screen and (max-width: map-get($grid-breakpoints, "md")) {
  h2 input {
    width: 90% !important;
  }
}

h2 input.form-control {
  text-align: center;
  font-size: calc(0.5rem + 2vmin + 1vw);
}

i.fa-print {
  color: gray;
  cursor: pointer;
  &:hover {
    color: $dark;
  }
}
.recipe {
  margin: 0 auto;
  border: 1px dashed gray;
  border-radius: 3px;
  padding: 15px;
  ul {
    list-style: none;
    li:before {
      content: "-";
      position: absolute;
      margin-left: -15px;
    }
  }
}
.multiplier {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 36px;
  color: gray;

  label {
    font-style: italic;
    padding-right: 15px;
  }

  input {
    font-size: 36px;
    border: none;
    border-bottom: 2px solid gray;
    border-radius: 0;
    text-align: center;
    width: 75px;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
  }

  input[type="number"]::-webkit-inner-spin-button,
  input[type="number"]::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
}

h2 {
  display: flex;
  align-items: center;
  justify-content: center;
}

textarea {
  min-height: 100px;
}

.subtle {
  color: gray;
  font-style: italic;
  font-size: 14px;
}

.recipe-title-display {
  min-height: 40px;
}

.instructions {
  white-space: pre-wrap;
}
</style>
