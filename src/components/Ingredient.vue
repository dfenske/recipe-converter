<template>
  <div>
    <div class="ingredient">
      {{ amount }} {{ unit }} {{ name }} <i v-if="showDelete" class="far fa-times-circle" @click="deleteItem"></i>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Ingredient',
  props: {
    name: String,
    amount: Number,
    unit: String,
    showDelete: Boolean
  },
  methods: {
    deleteItem() {
      this.$emit('deleteItem', this.name);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>

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

</style>
