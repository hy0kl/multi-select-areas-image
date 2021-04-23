<template>
  <div id="app">
    <h1>{{ msg }}</h1>
    <div>
      <multi-select-areas-image ref="hot-area-marker" :area="hotArea" @change="handelChange" />
    </div>
    <div v-for="(item, index) in hotArea" :key="'' + index" style="padding: 10px">
      x: <input :value="item.x" disabled />
      y: <input :value="item.y" disabled />
      w: <input :value="item.width" disabled />
      h: <input :value="item.height" disabled />
      content: <input v-model="item.op_content" @blur="reRender" />
    </div>
  </div>
</template>

<script>
import MultiSelectAreasImage from './components/multi-select-areas-image'

export default {
  name: 'app',
  components: { MultiSelectAreasImage },
  data () {
    return {
      msg: 'multi-select-areas-image',
      hotArea: []
    }
  },

  methods: {
    handelChange(data) {
      console.log({ data })
      this.hotArea = data
    },

    reRender(e) {
      console.log({ f: 'reRender', e })
      this.$refs['hot-area-marker'].draw()
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
