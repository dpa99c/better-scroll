<template>
  <div class="container">
    <h3>Wheel Performance Test</h3>
    <p>Items per wheel: {{itemCount}} | Wheels: {{wheelCount}} | Total DOM items: {{itemCount * wheelCount}}</p>
    <p id="perf-output" style="font-family: monospace; font-size: 12px; white-space: pre;"></p>
    <div class="picker-content">
      <div class="mask-top border-bottom-1px"></div>
      <div class="mask-bottom border-top-1px"></div>
      <div class="wheel-wrapper" ref="wheelWrapper">
        <div class="wheel" v-for="(data, wIdx) in wheelData" :key="wIdx">
          <ul class="wheel-scroll">
            <li
              v-for="(item, index) in data" :key="index"
              class="wheel-item">{{item.text}}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from '@better-scroll/core'
  import Wheel from '@better-scroll/wheel'
  BScroll.use(Wheel)

  const WHEEL_COUNT = 5
  const ITEM_COUNT = 90 // Matches mystery-walks expanded wheel data

  function generateItems(count, suffix) {
    const items = []
    for (let i = 0; i < count; i++) {
      items.push({ text: `${i}${suffix}`, value: i })
    }
    return items
  }

  export default {
    name: 'perf-test',
    data() {
      return {
        wheelCount: WHEEL_COUNT,
        itemCount: ITEM_COUNT,
        wheels: [],
        wheelData: [
          generateItems(ITEM_COUNT, '°'),
          generateItems(60, "'"),
          generateItems(60, '"'),
          generateItems(100, '.'),
          ['N','NE','E','SE','S','SW','W','NW'].map((d, i) => ({ text: d, value: i }))
        ]
      }
    },
    mounted() {
      this.$nextTick(() => {
        this._createWheels()
        this._startPerfMonitor()
      })
    },
    methods: {
      _createWheels() {
        const wrapper = this.$refs.wheelWrapper
        const wheels = wrapper.querySelectorAll('.wheel')

        for (let i = 0; i < wheels.length; i++) {
          const wheel = new BScroll(wheels[i], {
            wheel: {
              selectedIndex: Math.floor(this.wheelData[i].length / 2),
              rotate: 25,
              wheelWrapperClass: 'wheel-scroll',
              wheelItemClass: 'wheel-item'
            },
            useTransition: false,
            probeType: 3
          })
          this.wheels.push(wheel)
        }
      },
      _startPerfMonitor() {
        let frameCount = 0
        let lastTime = performance.now()
        let maxFrameTime = 0
        let frameDrops = 0
        const output = document.getElementById('perf-output')

        const measure = () => {
          const now = performance.now()
          const delta = now - lastTime
          frameCount++

          if (delta > 33) frameDrops++
          if (delta > maxFrameTime) maxFrameTime = delta
          lastTime = now

          if (frameCount % 60 === 0) {
            const fps = (1000 / (delta || 1)).toFixed(1)
            const info = `FPS: ~${fps} | Drops(>33ms): ${frameDrops} | MaxFrame: ${maxFrameTime.toFixed(1)}ms`
            if (output) output.textContent = info
          }

          requestAnimationFrame(measure)
        }
        requestAnimationFrame(measure)
      }
    },
    beforeDestroy() {
      this.wheels.forEach(w => w.destroy())
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  h3
    margin 10px 0
    text-align center
    font-size 16px
  p
    margin 5px 10px
    font-size 13px

  .border-bottom-1px, .border-top-1px
    position: relative
    &:before, &:after
      content: ""
      display: block
      position: absolute
      transform-origin: 0 0
  .border-bottom-1px
    &:after
      border-bottom: 1px solid #ebebeb
      left: 0
      bottom: 0
      width: 100%
      transform-origin: 0 bottom
  .border-top-1px
    &:before
      border-top: 1px solid #ebebeb
      left: 0
      top: 0
      width: 100%
      transform-origin: 0 top
  .picker-content
    position: relative
    top: 20px
    .mask-top, .mask-bottom
      z-index: 10
      width: 100%
      height: 68px
      pointer-events: none
      transform: translateZ(0)
    .mask-top
      position: absolute
      top: 0
      background: linear-gradient(to top, rgba(255, 255, 255, 0.4), rgba(255, 255, 255, 0.8))
    .mask-bottom
      position: absolute
      bottom: 1px
      background: linear-gradient(to bottom, rgba(255, 255, 255, 0.4), rgba(255, 255, 255, 0.8))
  .wheel-wrapper
    display: flex
    padding: 0 16px
    .wheel
      flex: 1
      width: 1%
      height: 173px
      overflow: hidden
      font-size: 18px
      .wheel-scroll
        padding: 0
        margin-top: 68px
        line-height: 36px
        list-style: none
        .wheel-item
          list-style: none
          height: 36px
          overflow: hidden
          white-space: nowrap
          color: #333
          text-align: center
</style>
</template>
