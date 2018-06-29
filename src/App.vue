<template>
  <div id="app">
    <nav class="navbar blue">
      <h1>Convert My Recipe</h1>
    </nav>
    <div class="container margin-bottom-lg margin-top-lg">
      <div class="full-width text-center margin-bottom-lg no-print">Hello! Enter your recipe's title, ingredients, and instructions and then enter the multiplier.</div>
      <input class="recipe-title full-width no-print form-control margin-sm" v-model.lazy="recipeTitle" placeholder="What is this recipe called?" />
      <div class="recipe original no-print">
        <div v-if="recipe.length === 0" class="subtle">Your recipe will show here. Enter ingredients below to start.</div>
        <div v-if="recipe.length > 0" class="subtle hover-text">Hover over an ingredient to delete it.</div>
        <ul>
          <li v-for="ingredient in recipe" :key="ingredient.name">
            <ingredient :name="ingredient.name" :amount="ingredient.amount" :unit="ingredient.unit" :isConvertedRecipe="false" v-on:delete-item="deleteItem" ></ingredient>
          </li>
        </ul>
      </div>
      <div class="add-ingredients no-print">
        <new-ingredient v-on:add-ingredient="addIngredient" />
      </div>
      <div class="full-width no-print">
        <textarea class="form-control margin-sm" placeholder="Enter the instructions here (optional)..." v-model.lazy="instructions" />
      </div>
      <div class="multiplier full-width padding-bottom-md margin-top-sm no-print">
        <label for="Multiplier">x </label>
        <input class="form-control margin-sm" v-model.number="multiplier" id="Multiplier" type="number"/>
      </div>
      
      <h2 class="full-width recipe-title-display">{{ recipeTitle ? recipeTitle + ' x ' + multiplier : ''}} </h2>
      <div class="full-width recipe">
        <div v-if="recipe.length > 0" class="subtle hover-text no-print">Hover over an ingredient to change the units.</div>
        <div class="text-right"><i class="fas fa-print fa-lg no-print" @click="print"></i></div>
        <ul>
          <li v-for="ingredient in convertedRecipe" :key="ingredient.name">
            <ingredient :name="ingredient.name" :amount="ingredient.amount" :unit="ingredient.unit" :isConvertedRecipe="true" v-on:update-ingredient="updateIngredient"></Ingredient>
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
  --base-font-size: 16px;
}
/* Grid stuff */
.app * {
  box-sizing: border-box;
  font-size: var(--base-font-size);
}
.container {
  --grid-template-num-columns: 1fr 1fr;
  display: grid;
  grid-template-columns: var(--grid-template-num-columns);
  grid-gap: 1em;
}
input.recipe-title {
  width: 100%;
  font-size: calc(0.5rem + 2vmin + 1vw);
  text-align: center;
}
.recipe.original {
  grid-column: var(--first-col);
}
.add-ingredients {
  grid-column: var(--second-col);
}
.full-width {
  grid-column: 1 / -1;
}
/* End grid stuff */

h2 input.form-control {
  text-align: center;
  font-size: 3em;
}
i.fa-print {
  color: gray;
  cursor: pointer;
  &:hover {
    color: $dark;
  }
}
.recipe {
  border: 1px dashed gray;
  border-radius: 3px;
  padding: 1em;
  ul {
    list-style: none;
    li:before {
      content: "-";
      position: absolute;
      margin-left: -1em;
    }
  }
  textarea {
    min-height: 100px;
  }
  .subtle {
    color: gray;
    font-style: italic;
    font-size: 1em;
  }
  .instructions {
    white-space: pre-wrap;
  }
}
.recipe-title-display {
  min-height: 2em;
  font-size: calc(0.5rem + 2vmin + 1vw);
}
.multiplier {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 3em;
  color: gray;
  border-bottom: 1px solid lightgray;

  label {
    font-style: italic;
    padding-right: 1em;
  }

  input {
    font-size: 1em;
    border: none;
    border-bottom: 2px solid gray;
    border-radius: 0;
    text-align: center;
    width: 2em;
    /* hide the arrows */
    &[type="number"]::-webkit-inner-spin-button,
    &[type="number"]::-webkit-outer-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }
  }
}

// mobile
@media screen and (max-width: map-get($grid-breakpoints, "md")) {
  .app {
    --base-font-size: calc(0.5rem + 2vmin + 1vw);
  }
  .container {
    --grid-template-num-columns: 1fr;
    --first-col: 1 / -1;
    --second-col: 1 / -1;
  }
  .recipe ul {
    margin-left: -1em;
  }
  i.fa-print {
    display: none;
  }
  .subtle.hover-text {
    display: none;
  }
}
</style>
