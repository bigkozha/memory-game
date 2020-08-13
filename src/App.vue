<template>
  <div id="app">
    <div class="center-area">
      <div class="board">
        <Loading v-if="isLoading"></Loading>
        <div v-for="(item, index) in items" :key="index" class="border-holder">
          <Tile :picId="item" @clicked="tileClicked" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Tile from './components/Tile.vue';
import Loading from './components/Loading.vue';

export default {
  name: 'App',
  components: {
    Tile,
    Loading,
  },
  data() {
    return {
      tileOpened: [],
      items: this.shuffle([1, 1, 2, 2, 3, 3, 4, 4, 5, 5 ]),
      isLoading: false,
    };
  },
  methods: {
    tileClicked(tile) {
      this.tileOpened.push(tile);
      if (this.tileOpened.length >= 2) {
        this.isLoading = true;
        setTimeout(() => {
          this.duringTimeout(tile);
        }, 2000);
      }
    },
    shuffle(a) {
      for (let i = a.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [a[i], a[j]] = [a[j], a[i]];
      }
      return a;
    },
    duringTimeout(tile) {
      this.tileOpened.forEach((t) => {
        t.show = false;
      });
      this.isLoading = false;
      this.hasWon(tile);
      this.tileOpened = [];
    },
    hasWon(tile) {
      if (
        tile.picId === this.tileOpened[0].picId &&
        tile._uid !== this.tileOpened[0]._uid
      ) {
        this.items = this.items.filter(function (obj) {
          return obj !== tile.picId;
        });
      }
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 10%;
}
* {
  box-sizing: border-box;
}

.center-area {
  margin-left: 25%;
  margin-right: auto;
}

.board {
  height: 60vmin;
  width: 60vmin;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr 1fr;
  grid-template-rows: repeat(3, 33.33%);
}

@media screen and (max-width: 600px) {
  .board {
    height: 90vmin;
  }
}

.cell-content-empty {
  width: 100%;
  height: 100%;
}
</style>
