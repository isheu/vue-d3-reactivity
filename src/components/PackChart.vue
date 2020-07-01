<template>
  <div>
    <h1 class="$style.text-labels">Circle Pack in D3</h1>
    <svg :height="height" :width="width">
      <g>
        <transition-group tag="g" name="fade">
          <circle
            v-for="c in circles"
            :key="c.idx"
            :r="c.r"
            :cx="c.x"
            :cy="c.y"
            :fill="c.fill"
            :stroke="c.stroke"
          ></circle>
        </transition-group>
        <transition-group tag="g" name="fade">
          <text v-for="c in circles" :key="c.idx" :x="c.x" :y="c.y">
            {{ c.idx }}
          </text>
        </transition-group>
        <!-- <slot name="no-data"></slot> -->
      </g>
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3'
import anime from 'animejs'

export default {
  name: 'PackChart',
  props: {
    data: { type: Array, required: true },
    height: { type: Number, default: 600 },
    width: { type: Number, default: 600 },
    padding: { type: Number, default: 10 },
    sizeDim: { type: Function, default: () => 1 },
    colorSet: {
      type: Array,
      default: () => ['#ff0000', '#00ff00', '#0000ff', '#c181e0']
    },
    animateAttr: { type: Array, default: () => ['r', 'x', 'y'] }
  },
  data() {
    return {
      circles: []
    }
  },
  computed: {
    colorScale() {
      return d3.scaleOrdinal().range(this.colorSet)
    },
    contentWidth() {
      return this.width - 2 * this.padding
    },
    contentHeight() {
      return this.height - 2 * this.padding
    },
    packedData() {
      const nestedData = d3
        .nest()
        .key(d => d.user)
        .entries(this.data)

      const packableData = { id: 'All Tweets', values: nestedData }
      return d3.hierarchy(packableData, d => d.values).sum(this.sizeDim)
    },
    flattenedData() {
      const packChart = d3
        .pack()
        .size([this.contentWidth, this.contentHeight])
        .padding(this.padding)
      return packChart(this.packedData).descendants()
    },
    flattenedDataWithFill() {
      return this.flattenedData.map(d =>
        Object.assign({}, d, { fill: this.colorScale(d.depth) })
      )
    }
  },
  watch: {
    flattenedDataWithFill() {
      this.transitionFromToData()
    },
  },
  methods: {
    transitionFromToData: function() {
      this.circles = this.flattenedDataWithFill.map(d => {
        let idx = ''
        switch (d.depth) {
          case 0:
            idx = d.data.id
            break
          case 1:
            idx = d.data.key
            break
          case 2:
            idx = `${d.data.user}-${d.data.timestamp}`
            break
        }
        const idxMatch = this.circles.findIndex(record => record.idx == idx)
        let record = {}
        this.animateAttr.forEach(attr => {
          if (attr == 'fill') {
            record[attr] = this.circles[idxMatch]
              ? this.circles[idxMatch][`to${attr.toUpperCase()}`]
              : '#c0c0c0'
          } else {
            record[attr] = this.circles[idxMatch]
              ? this.circles[idxMatch][`to${attr.toUpperCase()}`]
              : 0
          }
          record[`to${attr.toUpperCase()}`] = d[attr]
        })
        return Object.assign({}, { idx }, record, { stroke: 'grey' })
      })
      let animateConfig = {
        targets: this.circles,
        easing: 'linear',
        duration: 800
      }
      this.animateAttr.forEach(attr => {
        try {
          animateConfig[attr] = el => [
            el[`${attr}`],
            el[`to${attr.toUpperCase()}`]
          ]
        } catch {
          console.error(
            'Indicated an attribute for animation that does not exist'
          )
        }
      })
      let animation = anime(animateConfig)
      animation.play()
    }
  }
}
</script>
<style module>
.text-labels {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 12px;
  text-anchor: middle;
  font-weight: 600;
}
.fade-enter-active,
.fade-leave-active {
  transition: all 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
.fade-enter-active {
  transition-delay: 0.5s;
}
</style>
