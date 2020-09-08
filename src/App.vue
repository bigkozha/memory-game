<template>
  <div id="app">
    <div class="center-area">
      <div class="board">
        <Loading v-if="isLoading"></Loading>
        <div v-for="(item, index) in items" :key="index" class="border-holder">
          <Tile :picId="item" :id="index" @clicked="tileClicked" ref="items" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
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
      isLoading: false,
      items: [],
      chatSocket: null,
    };
  },
  methods: {
    tileClicked(tile) {
      this.tileOpened.push(tile);
      this.chatSocket.send(
        JSON.stringify({ tileOpened: this.tileOpened.map((t) => t.id) })
      );
      if (this.tileOpened.length >= 2) {
        this.duringTimeout();
      }
    },
    duringTimeout() {
      this.isLoading = true;
      setTimeout(() => {
        this.$refs.items.forEach((t) => {
          t.show = false;
        });
        this.isLoading = false;
        this.checkWin();
        this.tileOpened = [];
      }, 2000);
    },
    checkWin() {
      if (
        this.tileOpened[0].picId === this.tileOpened[1].picId &&
        this.tileOpened[0].id !== this.tileOpened[1].id
      ) {
        this.$refs.items.forEach((t) => {
          if (t.picId === this.tileOpened[0].picId) {
            t.hasWon = true;
            t.show = true;
          }
        });
      }
    },
  },
  mounted() {
    this.chatSocket = new WebSocket(
      'ws://' + '127.0.0.1:8000' + '/ws/chat/' + 'myroom' + '/'
    );

    this.chatSocket.onmessage = (e) => {
      const data = JSON.parse(e.data);
      if (data.items) {
        this.items = data.items;
      }
      if (data.tile_opened) {
        const tiles = data.tile_opened;
        if (tiles.length >= 2) {
          this.$refs.items.forEach((t) => {
            if (t.id === tiles[0] || t.id === tiles[1]) {
              t.show = true;
            }
          });

          this.isLoading = true;
          setTimeout(() => {
            this.$refs.items.forEach((t) => {
              t.show = false;
            });
            this.isLoading = false;
            if (
              this.$refs.items.find((i) => i.id === tiles[0]).picId ===
                this.$refs.items.find((i) => i.id === tiles[1]).picId &&
              tiles[0] !== tiles[1]
            ) {
              this.$refs.items.forEach((t) => {
                if (
                  t.picId ===
                  this.$refs.items.find((i) => i.id === tiles[0]).picId
                ) {
                  t.hasWon = true;
                  t.show = true;
                }
              });
            }
            this.tileOpened = [];
          }, 2000);
        }
      }
    };

    this.chatSocket.onclose = () => {
      console.error('Chat socket closed unexpectedly');
    };
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
