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
      </g>
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3'
import { gsap } from 'gsap'
export default {
  name: 'PackChart',
  props: {
    data: { type: Array, required: true },
    height: { type: Number, default: 600 },
    width: { type: Number, default: 600 },
    padding: { type: Number, default: 10 },
    sizeDim: { type: Function, default: () => 1 },
    colorSet: { type: Array, default: () => ['green', 'blue', 'yellow', 'red'] }
  },
  data() {
    return {
      animateAttr: ['r', 'x', 'y'],
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
    }
  },
  watch: {
    flattenedData() {
      this.transitionFromToData()
    },
    colorScale() {
      this.transitionFromToData()
    }
  },
  methods: {
    transitionFromToData: function() {
      this.circles = this.flattenedData.map(d => {
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
        const fill = this.colorScale(d.depth)
        let record = {}
        this.animateAttr.forEach(attr => {
          record[attr] = this.circles[idxMatch]
            ? this.circles[idxMatch][`to${attr.toUpperCase()}`]
            : 0
          record[`to${attr.toUpperCase()}`] = d[attr]
        })
        return Object.assign({}, { fill, idx }, record, { stroke: 'grey' })
      })
      let fromSpec = {}
      let toSpec = {}
      this.animateAttr.forEach(attr => {
        try {
          fromSpec[attr] = (i, d) => d[`${attr}`]
          toSpec[attr] = (i, d) => d[`to${attr.toUpperCase()}`]
        } catch {
          console.warn(
            'Attempted to customize an attribute for animation that is not x, y, or r'
          )
        }
      })
      gsap.fromTo(this.circles, fromSpec, toSpec)
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
