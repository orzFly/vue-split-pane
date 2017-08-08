<template>
  <div :style="{ cursor, userSelect}" class="vue-splitter-container clearfix" @mouseup="onMouseUp" @mousemove="onMouseMove">

    <div class="resizer-wrap" :class="split" :style="{ [type]: percent, ['min-' + type]: min, ['max-' + type]: max}">
      <resizer :style="{ [resizeType]: '100%'}" :split="split" @mousedown.native="onMouseDown"></resizer>
    </div>

    <pane ref="paneL" class="splitter-pane splitter-paneL" :split="split" :style="{ [type]: percent, ['min-' + type]: min, ['max-' + type]: max}">
      <slot name="paneL"></slot>
    </pane>

    <pane class="splitter-pane splitter-paneR" :split="split" :style="{ [type]: 'calc(100% - ' + percent + ')', ['min-' + type]: 'calc(100% - ' + max + ')', ['max-' + type]: 'calc(100% - ' + min + ')'}">
      <slot name="paneR"></slot>
    </pane>

  </div>
</template>

<script>
  import Resizer from './resizer.vue';
  import Pane from './pane.vue';

  export default {
    name: 'splitPane',
    components: { Resizer, Pane },
    props: {
      min: {
        type: String,
        default: "10%"
      },
      max: {
        type: String,
        default: "90%"
      },
      default: {
        type: String,
        default: "50%"
      },
      split: {
        validator(value) {
          return ['vertical', 'horizontal'].indexOf(value) >= 0
        },
        required: true
      }
    },
    computed: {
      userSelect() {
        return this.active ? 'none' : ''
      },
      cursor() {
        return this.active ? 'col-resize' : ''
      }
    },
    data() {
      return {
        active: false,
        hasMoved: false,
        height: null,
        percent: this.default,
        paneLsize: this.default,
        type: this.split === 'vertical' ? 'width' : 'height',
        resizeType: this.split === 'vertical' ? 'left' : 'top'
      }
    },
    methods: {
      onMouseDown() {
        this.active = true;
        this.hasMoved = false;
      },
      onMouseUp() {
        this.active = false;
      },
      onMouseMove(e) {
        if (e.buttons === 0 || e.which === 0) {
          this.active = false;
        }
        if (this.active) {
          let offset = 0;
          let target = e.currentTarget;
          if (this.split === 'vertical') {
            while (target) {
              offset += target.offsetLeft;
              target = target.offsetParent;
            }
          } else {
            while (target) {
              offset += target.offsetTop;
              target = target.offsetParent;
            }
          }
          const currentPage = this.split === 'vertical' ? e.pageX : e.pageY;
          const targetOffset = this.split === 'vertical' ? e.currentTarget.offsetWidth : e.currentTarget.offsetHeight;
          this.percent = (currentPage - offset) + "px";
          this.$emit('resize');
          this.hasMoved = true;
        }
      }
    }
  }
</script>

<style scoped>
.resizer-wrap.vertical {
    position: absolute;
    left: 0px;
    height: 100%;
}

.resizer-wrap.horizontal {
    position: absolute;
    top: 0px;
    width: 100%;
}
.clearfix:after {
  visibility: hidden;
  display: block;
  font-size: 0;
  content: " ";
  clear: both;
  height: 0;
}

.vue-splitter-container {
  height: 100%;
  position: relative;
}
</style>
