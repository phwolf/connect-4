<template>
  <div 
    v-bind:class="[state === 0 || state === 1 ? 'tileclicked' : 'tile']"
    v-bind:style="{
      'background-color': state === 0 || state === 1 ? colors[state] : 'white',
      'border': win ? 'solid 1.5px red' : 'solid 1.5px white'
    }"
    v-on:click="handleClick"
    >
  </div>
</template>

<script>
export default {
  name: 'disc-tile',
  props: ['state', 'win', 'index'],
  data () {
    return {
      colors: ['orange', 'steelblue']
    }
  },
  methods: {
    handleClick() {
      if (this.state == null) {
        console.log(this.index)
        this.$emit('clicked', this.index);
      }
    }
  }
}
</script>

<style>
  .tile {
    border-radius: 25px;
    height: 50px;
    width: 50px;
    cursor: pointer;
  }
  .tileclicked {
    border-radius: 25px;
    height: 50px;
    width: 50px;
  }

</style>
