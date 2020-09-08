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
import Tile from './components/Tile.vue';
import Loading from './components/Loading.vue';

const TILE_TIMEOUT = 2000;
const SERVER_ADDRESS = '127.0.0.1:8000';

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
        this.showAllTiles();
        this.checkWin();
        this.tileOpened = [];
      }, TILE_TIMEOUT);
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
        this.chatSocket.send(
          JSON.stringify({ tile_won: this.tileOpened.map((t) => t.id) })
        );
      }
    },
    setOpenedTiles(tiles) {
      if (tiles.length >= 2) {
        const localItems = this.$refs.items.filter(
          (i) => i.id === tiles[0] || i.id === tiles[1]
        );

        localItems.forEach((t) => {
          t.show = true;
        });

        this.isLoading = true;

        setTimeout(() => {
          this.showAllTiles();

          if (
            localItems[0].picId === localItems[1].picId &&
            tiles[0] !== tiles[1]
          ) {
            this.$refs.items.forEach((t) => {
              if (t.picId === localItems[0].picId) {
                t.hasWon = true;
                t.show = true;
              }
            });
          }
          this.tileOpened = [];
        }, TILE_TIMEOUT);
      }
    },
    setTileWon(tilewon) {
      this.$nextTick(() => {
        let self = this;
        tilewon.forEach((w) => {
          const won = self.$refs.items.find((i) => i.id === w);
          won.hasWon = true;
          won.show = true;
        });
      });
    },
    showAllTiles() {
      this.$refs.items.forEach((t) => {
        t.show = false;
      });
      this.isLoading = false;
    },
  },
  mounted() {
    this.chatSocket = new WebSocket(
      'ws://' + SERVER_ADDRESS + '/ws/game/' + 'myroom' + '/'
    );

    this.chatSocket.onmessage = (e) => {
      const data = JSON.parse(e.data);
      if (data.items) {
        this.items = data.items;
      }
      if (data.tile_won) {
        this.setTileWon(data.tile_won);
      }
      if (data.tile_opened) {
        this.setOpenedTiles(data.tile_opened);
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
  text-align: center;
  margin-top: 10%;
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
</style>
