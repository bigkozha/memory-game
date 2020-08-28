<template>
  <img @click="clicked()" alt="Vue logo" :src="src()" />
</template>

<script>
export default {
  name: 'Tile',
  props: {
      picId: Number,
  },
  data() {
    return {
      show: false,
      hasWon: false,
    };
  },
  methods: {
    clicked() {
      this.show ? (this.show = false) : (this.show = true);
      this.$emit('clicked', this);
    },
    src() {
      if (this.hasWon) {
        return require('../assets/' + this.picId + '.png');
      }
      return this.show
        ? require('../assets/' + this.picId + '.png')
        : require('../assets/back.png');
    },
  },
};
</script>