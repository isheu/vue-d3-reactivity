<template>
  <div id="app">
    <PackChart
      :data="loadData"
      :color-set="colorSet"
      :size-dim="nodeSizeFn"
      :animate-attr="attrToAnimate"
    />
    <button @click="changeData">Change Data</button>
    <button @click="changeColor">Change Color</button>
  </div>
</template>

<script>
import PackChart from '@/components/PackChart.vue'

export default {
  name: 'app',
  components: {
    PackChart
  },
  data: function() {
    return {
      loadData: [],
      colorSet: undefined,
      attrToAnimate: ['r', 'x', 'y', 'fill'],
      nodeSizeFn: d =>
        d.retweets ? d.retweets.length + d.favorites.length + 1 : 1
    }
  },
  mounted() {
    this.fetchData()
  },
  methods: {
    changeData() {
      this.loadData = this.loadData.filter((d, i) => i > 3)
      this.colorSet = ['#c0ffc0', '#cccccc', '#ffc0c0', '#c0c0ff']
    },
    changeColor() {
      this.colorSet = ['#c0ffc0', '#cccccc', '#ffc0c0', '#c0c0ff']
    },
    fetchData() {
      let data = [
        {
          user: 'Al',
          content: 'I really love seafood.',
          timestamp: ' Mon Dec 23 2013 21:30 GMT-0800 (PST)',
          retweets: ['Raj', 'Pris', 'Roy'],
          favorites: ['Sam']
        },
        {
          user: 'Al',
          content: "I take that back, this doesn't taste so good.",
          timestamp: 'Mon Dec 23 2013 21:55 GMT-0800 (PST)',
          retweets: ['Roy'],
          favorites: []
        },
        {
          user: 'Al',
          content: "From now on, I'm only eating cheese sandwiches.",
          timestamp: 'Mon Dec 23 2013 22:22 GMT-0800 (PST)',
          retweets: [],
          favorites: ['Roy', 'Sam']
        },
        {
          user: 'Roy',
          content: 'Great workout!',
          timestamp: ' Mon Dec 23 2013 7:20 GMT-0800 (PST)',
          retweets: [],
          favorites: []
        },
        {
          user: 'Roy',
          content: 'Spectacular oatmeal!',
          timestamp: ' Mon Dec 23 2013 7:23 GMT-0800 (PST)',
          retweets: [],
          favorites: []
        },
        {
          user: 'Roy',
          content: 'Amazing traffic!',
          timestamp: ' Mon Dec 23 2013 7:47  GMT-0800 (PST)',
          retweets: [],
          favorites: []
        },
        {
          user: 'Roy',
          content: 'Just got a ticket for texting and driving!',
          timestamp: ' Mon Dec 23 2013 8:05 GMT-0800 (PST)',
          retweets: [],
          favorites: ['Sam', 'Sally', 'Pris']
        },
        {
          user: 'Pris',
          content: 'Going to have some boiled eggs.',
          timestamp: ' Mon Dec 23 2013 18:23 GMT-0800 (PST)',
          retweets: [],
          favorites: ['Sally']
        },
        {
          user: 'Pris',
          content: 'Maybe practice some gymnastics.',
          timestamp: ' Mon Dec 23 2013 19:47  GMT-0800 (PST)',
          retweets: [],
          favorites: ['Sally']
        },
        {
          user: 'Sam',
          content: "@Roy Let's get lunch",
          timestamp: ' Mon Dec 23 2013 11:05 GMT-0800 (PST)',
          retweets: ['Pris'],
          favorites: ['Sally', 'Pris']
        }
      ]
      this.loadData = data
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
</style>
