<template>
  <div class="fit-layout-wrapper" :style="wrapperStyle">
    <div class="fit-layout" :style="appStyle">
      <slot />
    </div>
  </div>
</template>

<script>
import { computed } from "vue"
import { useWindowResize } from "./resize-hook"

const DesignSize = {
  width: 3840,
  height:2160,
}

const FIT_MODE = {
  FIT_HEIGHT: "fitHeight",
  FIT_WIDTH: "fitWidth",
  STRETCH: "stretch",
  ORIGIN: "origin",
}

const getViewScale = (userDesignSize, mode, containerSize) => {
  switch (mode) {
    case FIT_MODE.FIT_WIDTH:
      const scaleW = containerSize.width / (userDesignSize || DesignSize).width
      return {
        x: scaleW,
        y: scaleW,
      }
    case FIT_MODE.FIT_HEIGHT:
      const scaleH = containerSize.height / (userDesignSize || DesignSize).height
      return {
        x: scaleH,
        y: scaleH,
      }
    case FIT_MODE.ORIGIN:
      return {
        x: 1,
        y: 1,
      }
    case FIT_MODE.STRETCH:
    default:
      return {
        x: containerSize.width / (userDesignSize || DesignSize).width,
        y: containerSize.height / (userDesignSize || DesignSize).height,
      }
  }
}

export default {
  name: "fit-layout",
  props: {
    width: {
      type: Number,
      default() {
        return DesignSize.width
      },
    },
    height: {
      type: Number,
      default() {
        return DesignSize.height
      },
    },
    fixHeight: {
      type: Number,
      default() {
        return null
      },
    },
    mode: {
      type: String,
      default() {
        return FIT_MODE.STRETCH
      },
    },
  },
  setup(props, context) {
    // 监听窗口变化
    const containerSize = useWindowResize()

    // 视图尺寸
    const viewSize = computed(() => ({
      width: props.width || DesignSize.width,
      height: props.height || DesignSize.height,
    }))

    // 视图缩放比例
    const viewScale = computed(() => {
      return getViewScale(
        {
          width: props.width || DesignSize.width,
          height: props.height || DesignSize.width,
        },
        props.mode,
        {
          width: containerSize.width.value,
          height: containerSize.height.value,
        }
      )
    })

    // 高度固定样式
    const outterHeightScaled = computed(() => {
      if (props.fixHeight != null) {
        return `${viewScale.value.y * props.fixHeight}px`
      } else {
        return null
      }
    })

    // 视图样式
    const appStyle = computed(() => {
      return {
        width: `${viewSize.value.width}px`,
        height:
          props.fixHeight != null
            ? `${props.fixHeight}px`
            : `${viewSize.value.height}px`,
        transform: `scale(${viewScale.value.x},${viewScale.value.y})`,
      }
    })

    // 容器样式
    const wrapperStyle = computed(() => {
      console.log("wrapperStyle", "outterHeightScaled", outterHeightScaled)
      switch (props.mode) {
        case FIT_MODE.FIT_WIDTH:
          return {
            display: "block",
            overflow: "hidden auto",
            // width: "100vw"
          }
        case FIT_MODE.FIT_HEIGHT:
          return {
            height: outterHeightScaled.value
              ? outterHeightScaled.value
              : "100%",
            boxSizing: "content-box",
            overflow: "auto hidden",
          }
        case FIT_MODE.ORIGIN:
          return {
            height: outterHeightScaled.value
              ? outterHeightScaled.value
              : "100vh",
            overflow: "auto",
          }
        case FIT_MODE.STRETCH:
        default:
          return {
            width: "100vw",
            height: outterHeightScaled.value
              ? outterHeightScaled.value
              : "100vh",
            // overflow: "hidden",
          }
      }
    })

    return {
      appStyle,
      wrapperStyle,
    }
  },
}
</script>

<style scoped>
.fit-layout-wrapper{
  overflow: hidden;
}
.fit-layout {
  transform-origin: left top;
  z-index: 1;
}
</style>
