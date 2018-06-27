<template>
  <div id="app">
    <nav class="navbar navbar-dark bg-dark">
      <h1 class="text-white">Recipe Converter</h1>
    </nav>

    <div class="container">
      <h2 class="no-print recipe-title"><input class="form-control title" v-model.lazy="recipeTitle" placeholder="What is this recipe called?" /></h2>
      <div class="row no-print">
        <div class="col-sm recipe">
          <ul>
            <li v-for="ingredient in recipe" :key="ingredient.name">
              <ingredient :name="ingredient.name" :amount="ingredient.amount" :unit="ingredient.unit" :converted="false" v-on:delete-item="deleteItem" ></Ingredient>
            </li>
          </ul>
        </div>
        <div class="col-sm">
          <new-ingredient v-on:add-ingredient="addIngredient" />
        </div>
      </div>
      <div class="multiplier mt-2 no-print">
        <label for="Multiplier">x </label>
        <input class="form-control" v-model.number="multiplier" id="Multiplier" type="number"/>
      </div>
      
      <hr class="no-print"/>
      <h2 class="minHeight40">{{ recipeTitle ? recipeTitle + ' x ' + multiplier : ''}} </h2>
      <div class="recipe">
        <div class="text-right"><i class="fas fa-print fa-lg print-icon no-print" @click="print"></i></div>
        <ul>
          <li v-for="ingredient in convertedRecipe" :key="ingredient.name">
            <ingredient :name="ingredient.name" :amount="ingredient.amount" :unit="ingredient.unit" :converted="true" v-on:update-ingredient="updateIngredient"></Ingredient>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import localforage from 'localforage';
import Ingredient from './components/Ingredient.vue';
import NewIngredient from './components/NewIngredient.vue';
import { volume, weight } from './assets/conversions.js';

export default {
  name: 'app',
  components: {
    Ingredient,
    NewIngredient
  },
  data() {
    return {
      recipeTitle: '',
      multiplier: 1,
      recipe: [],
      units: {
        volume,
        weight
      }
    };
  },
  async mounted() {
    localforage.getItem('recipe', (err, value) => this.mountRecipe(err, value));
    localforage.getItem('recipeTitle', (err, value) => this.mountTitle(err, value));
    localforage.getItem('multiplier', (err, value) => this.mountMultiplier(err, value));
  },
  methods: {
    mountRecipe(err, value) {
      if (err || !value) {
        localforage.setItem('recipe', []);
        this.recipe = [];
      } else {
        this.recipe = value;
      }
    },
    mountTitle(err, value) {
      if (err || !value) {
        localforage.setItem('recipeTitle', []);
        this.recipeTitle = [];
      } else {
        this.recipeTitle = value;
      }
    },
    mountMultiplier(err, value) {
      if (err || !value) {
        localforage.setItem('multiplier', []);
        this.multiplier = [];
      } else {
        this.multiplier = value;
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
      this.recipe.splice(this.recipe.findIndex(x => x.name===name), 1);
    },
    updateIngredient(name, newAmount, newUnit) {
      const index = this.recipe.findIndex(x => x.name === name);
      this.recipe.splice(index, 1, { name: name, amount: newAmount, unit: newUnit });
    },
    print() {
      window.print();
    }
  },
  watch: {
    recipe: function() {
      localforage.setItem('recipe', this.recipe);
    },
    multiplier: function() {
      localforage.setItem('multiplier', this.multiplier);
    },
    recipeTitle: function() {
      localforage.setItem('recipeTitle', this.recipeTitle);
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
}
</script>

<style scoped lang='scss'>
.multiplier {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 36px;
  color: gray;

  label {
    margin: 0;
    font-style: italic;
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

  input[type=number]::-webkit-inner-spin-button, 
  input[type=number]::-webkit-outer-spin-button { 
    -webkit-appearance: none; 
    margin: 0; 
  }
}

.recipe-title {
  display: flex;
  align-items: center;
  justify-content: center;  
}

.form-control.title {
  width: 50%;
  text-align: center;
  font-size: 36px;
}

.minHeight40 {
  min-height: 40px;
}
</style>
