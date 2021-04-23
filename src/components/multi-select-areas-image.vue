<template>
  <div class="vicp-wrap">
    <img :src="imgSrc" :width="width" :height="height" alt="" class="no-selected" onselectstart="return false">
    <canvas
      ref="canvas"
      class="cnt-box"
      :width="width"
      :height="height"
      @mousemove="mousemove"
      @mousedown="mousedown"
      @mouseup="mouseup"
      @dblclick="dblclick"
    />
  </div>
</template>

<script>
'use strict'
// https://blog.gongshw.com/jquery.areaSelect.js/demo/demo.html
// https://github.com/gongshw/jquery.areaSelect.js
export function near(point1, point2, s) {
  return Math.pow(point1.x - point2.x, 2) + Math.pow(point1.y - point2.y, 2) <= Math.pow(s, 2)
}

export function get_offset_X(event) {
  return event.offsetX ? event.offsetX : event.originalEvent.layerX
}

export function get_offset_Y(event) {
  return event.offsetY ? event.offsetY : event.originalEvent.layerY
}

export function setAreaDirection(area, direction) {
  if (area !== undefined && direction !== undefined) {
    const x1 = area.x
    const x2 = area.x + area.width
    const y1 = area.y
    const y2 = area.y + area.height
    const width = Math.abs(area.width)
    const height = Math.abs(area.height)
    const minOrMax = { '1': Math.min, '-1': Math.max }
    area.x = minOrMax[direction.x](x1, x2)
    area.y = minOrMax[direction.y](y1, y2)
    area.width = direction.x * width
    area.height = direction.y * height
  }
}

export default {
  name: 'MultiSelectAreasImage',
  props: {
    imgSrc: {
      type: String,
      default: 'https://blog.gongshw.com/jquery.areaSelect.js/demo/saber.jpg'
    },
    // 剪裁图片的宽
    width: {
      type: Number,
      default: 800
    },
    // 剪裁图片的高
    height: {
      type: Number,
      default: 600
    },
    areas: {
      type: Array,
      default() {
        return []
      }
    }
  },

  data() {
    // const { width, height, imgSrc } = this

    return {
      options: {
        deleteMethod: 'doubleClick', // or doubleClick
        padding: 3,
        area: { strokeStyle: 'red', lineWidth: 2 },
        point: { size: 3, fillStyle: 'black' }
      },
      canvas: null,
      canvasCtx: null,
      status: '',
      dragging: false,
      resizeDirection: null,
      dragAreaOffset: {},
      moveDownPoint: {},
      currentArea: undefined,
      AreaSelectStatus: {
        CREATE: 'create',
        MOVE: 'move',
        RESIZE: 'resize',
        NEAR: 'near'
      },
      Direction: {
        NE: { name: 'NE', x: 1, y: -1, cursor: 'nesw-resize' },
        NW: { name: 'NW', x: -1, y: -1, cursor: 'nwse-resize' },
        SE: { name: 'SE', x: 1, y: 1, cursor: 'nwse-resize' },
        SW: { name: 'SW', x: -1, y: 1, cursor: 'nesw-resize' }
      },
      DeleteMethod: { CLICK: 'click', DOUBLE_CLICK: 'doubleClick' }
    }
  },

  computed: {
  },

  watch: {
  },

  created() {
  },

  mounted() {
    this.canvas = this.$refs.canvas
    this.canvasCtx = this.$refs.canvas.getContext('2d')
  },

  destroyed() {
  },

  methods: {
    init(data) {
      console.log({ f: 'init', data })
      this.areas = data
      if (this.areas.length > 0) {
        this.draw()
      }
    },

    mousemove(event) {
      // console.log({ event })
      const offsetX = get_offset_X(event)
      const offsetY = get_offset_Y(event)
      if (this.dragging) {
        this.onDragging(offsetX, offsetY)
      } else {
        this.onMouseMoving(offsetX, offsetY)
      }
    },

    mousedown(event) {
      this.moveDownPoint = { x: get_offset_X(event), y: get_offset_Y(event) }
      this.onDragStart(get_offset_X(event), get_offset_Y(event))
    },

    mouseup(event) {
      if (get_offset_X(event) === this.moveDownPoint.x && get_offset_Y(event) === this.moveDownPoint.y) {
        this.onClick(get_offset_X(event), get_offset_Y(event))
      }
      this.onDragStop()
    },

    dblclick(event) {
      this.onDoubleClick(get_offset_X(event), get_offset_Y(event))
    },

    getArea(x, y, padding) {
      padding = padding === undefined ? 0 : padding
      for (const index in this.areas) {
        const area = this.areas[index]
        const abs = Math.abs
        const x1 = area.x
        const x2 = area.x + area.width
        const y1 = area.y
        const y2 = area.y + area.height

        if (padding >= 0 && abs(x1 - x) + abs(x2 - x) - abs(area.width) <= padding * 2 &&
          abs(y1 - y) + abs(y2 - y) - abs(area.height) <= padding * 2) {
          return area
        }

        if (padding < 0 && abs(x1 - x) + abs(x2 - x) - abs(area.width) === 0 &&
          abs(y1 - y) + abs(y2 - y) - abs(area.height) === 0 &&
          abs(abs(x1 - x) - abs(x2 - x)) <= abs(area.width) + 2 * padding &&
          abs(abs(y1 - y) - abs(y2 - y)) <= abs(area.height) + 2 * padding) {
          return area
        }
      }

      return undefined
    },

    getPositionPoints(area) {
      const points = {}
      for (const d in this.Direction) {
        points[d] = {
          x: area.x + area.width * (this.Direction[d].x + 1) / 2,
          y: area.y + area.height * (this.Direction[d].y + 1) / 2
        }
      }
      return points
    },

    onDragging(x, y) {
      const area = this.currentArea
      switch (this.status) {
        case this.AreaSelectStatus.RESIZE:
          area.width = x - area.x
          area.height = y - area.y
          break

        case this.AreaSelectStatus.MOVE:
          area.x = (x + this.dragAreaOffset.x)
          area.y = (y + this.dragAreaOffset.y)
          break

        case this.AreaSelectStatus.CREATE:
          break
      }

      this.draw()
    },

    onMouseMoving(x, y) {
      const area = this.getArea(x, y, this.options.padding)
      const canvas = this.canvas
      // console.log({ canvas })

      if (area !== undefined) {
        this.currentArea = area
        let nearDrag = false
        let dragDirection = null
        const dragPoints = this.getPositionPoints(area)
        for (const d in dragPoints) {
          if (near({ x: x, y: y }, dragPoints[d], this.options.padding)) {
            nearDrag = true
            dragDirection = this.Direction[d]
            break
          }
        }
        if (nearDrag) {
          canvas.style.cursor = dragDirection.cursor
          this.status = this.AreaSelectStatus.RESIZE
          this.resizeDirection = dragDirection
        } else if (this.getArea(x, y, -this.options.padding) !== undefined) {
          canvas.style.cursor = 'move'
          this.status = this.AreaSelectStatus.MOVE
        } else {
          canvas.style.cursor = 'auto'
          this.status = this.AreaSelectStatus.NEAR
        }
      } else {
        this.currentArea = undefined
        canvas.style.cursor = 'default'
        this.status = this.AreaSelectStatus.CREATE
      }
      this.draw()
    },

    onDragStart(x, y) {
      let newArea
      this.dragging = true
      switch (this.status) {
        case this.AreaSelectStatus.RESIZE:
          !this.currentArea || setAreaDirection(this.currentArea, this.resizeDirection)
          break

        case this.AreaSelectStatus.MOVE:
          this.dragAreaOffset = { x: this.currentArea.x - x, y: this.currentArea.y - y }
          break

        case this.AreaSelectStatus.CREATE:
          newArea = {
            x: x,
            y: y,
            width: 0,
            height: 0,
            op_type: '',
            op_content: ''
          }
          this.areas.push(newArea)
          this.currentArea = newArea
          this.status = this.AreaSelectStatus.RESIZE
          break

        default:
          break
      }
    },

    onClick(x, y) {
      const area = this.getArea(x, y, this.options.padding)
      if (area !== undefined && this.options.deleteMethod === this.DeleteMethod.CLICK) {
        this.deleteArea(area)
        this.draw()
      }
    },

    onDoubleClick(x, y) {
      const area = this.getArea(x, y, this.options.padding)
      if (area !== undefined && this.options.deleteMethod === this.DeleteMethod.DOUBLE_CLICK) {
        this.deleteArea(area)
        this.draw()
      }
    },

    onDragStop() {
      this.dragging = false
      switch (this.status) {
        case this.AreaSelectStatus.RESIZE:
          if (this.currentArea !== undefined) {
            if (this.currentArea.width === 0 && this.currentArea.height === 0) {
              this.deleteArea(this.currentArea)
              this.currentArea = undefined
              this.status = this.AreaSelectStatus.CREATE
            } else {
              setAreaDirection(this.currentArea, this.Direction.SE)
              this.triggerChange()
            }
          }
          break

        case this.AreaSelectStatus.MOVE:
          this.triggerChange()
          break
      }
    },

    deleteArea(area) {
      const areas = this.areas
      const index = areas.indexOf(area)
      if (index >= 0) {
        areas.splice(areas.indexOf(area), 1)
        this.currentArea = undefined
        this.triggerChange()
        this.status = this.AreaSelectStatus.CREATE
      }
    },

    draw() {
      const g2d = this.canvasCtx

      /* clear canvas */
      g2d.clearRect(0, 0, this.width, this.height)

      /* draw areas */
      g2d.strokeStyle = this.options.area.strokeStyle
      g2d.lineWidth = this.options.area.lineWidth
      for (const index in this.areas) {
        const area = this.areas[index]
        g2d.strokeRect(area.x, area.y, area.width, area.height)

        g2d.font = 'normal 20px arial'
        g2d.fillStyle = 'black'
        g2d.fillText(area.op_content, area.x + 10, area.y + area.height - 10)
      }

      /* draw current area */
      const area = this.currentArea
      g2d.fillStyle = this.options.point.fillStyle
      if (area !== undefined) {
        const positionPoints = this.getPositionPoints(area)
        /* draw position point */
        for (const index in positionPoints) {
          const point = positionPoints[index]
          g2d.beginPath()
          g2d.arc(point.x, point.y, this.options.point.size, 0, Math.PI * 2, true)
          g2d.closePath()
          g2d.fill()
        }
      }
    },

    triggerChange() {
      this.$emit('change', this.areas)
    }
  }
}
</script>

<style scoped>
.vicp-wrap {
  text-align: left;
  position: relative;
}
.cnt-box{
  position: absolute;
  z-index: 999999;
  top: 0;
  left: 0;
}
.no-selected {
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
  -khtml-user-select: none;
  user-select: none;
}
</style>
