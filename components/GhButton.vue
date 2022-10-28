<template>
  <div
    :class="[
      disabled ? 'GhButton-disabled' : 'GhButton-normal',
      onlyIcon ? 'GhButton-onlyIcon' : '',
    ]"
    @click="clickHandler"
  >
    <div
      :class="[
        'GhButton',
        `GhButton-size-${size}`,
        `GhButton-iconAlign-${iconAlign}`,
        `GhButton-${theme}-${color}`,
      ]"
      :style="propsStyle"
    >
      <i v-if="icon" :class="[iconStore, icon]" />
      <p :style="propsTextStyle" v-if="!onlyIcon">{{ text }}</p>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    text: {
      type: String,
      default: 'button'
    },
    minWidth: {
      type: Number,
      default: 0
    },
    normalFontColor: {
      type: String,
      default: ''
    },
    cusStyle: {
      type: Object
      // default: {}
    },
    disabled: {
      type: Boolean,
      default: false
    },
    size: {
      type: String,
      default: 'small'
      // medium, small, mini
    },
    theme: {
      type: String,
      default: 'fill'
      // fill, solid, dashed
    },
    color: {
      type: String,
      default: 'blue'
      // blue, red
    },
    icon: {
      type: String,
      default: ''
      // iconfont库中的icon Class
    },
    iconStore: {
      type: String,
      default: 'iconfont'
      // iconfont库的Class
    },
    iconAlign: {
      type: String,
      default: 'left'
      // left, right
    },
    onlyIcon: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    minWidthPX() {
      return this.minWidth > 0 ? `${this.minWidth}px` : null
    },
    propsStyle() {
      const style = { ...this.cusStyle }
      if (this.minWidthPX) style['minWidth'] = this.minWidthPX
      return style
    },
    propsTextStyle() {
      const style = { }
      console.log('字体颜色===>', this.normalFontColor)
      if (this.normalFontColor) style['color'] = this.normalFontColor
      return style
    }
  },
  methods: {
    clickHandler() {
      if (this.disabled) return
      this.$emit('click')
    }
  }
}
</script>

<style lang="scss" scoped>
.GhButton {
  width: fit-content;
  display: flex;
  border-radius: 4px;
  cursor: pointer;
  transition: 0.2s all;
  border-width: 1px;
  align-items: center;
  i {
    font-size: 24px;
  }
  p {
    width: 100%;
    text-align: center;
    font-family: "Source Han Sans CN";
    font-size: 16px;
    line-height: 24px;
    user-select: none;
    transition: 0.2s all;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
}
.GhButton-size-medium {
  padding: 12px 16px;
}
.GhButton-size-small {
  padding: 8px 12px;
}
.GhButton-size-mini {
  padding: 4px 8px;
}
.GhButton-iconAlign-left {
  flex-direction: row;
  i {
    margin-right: 8px;
  }
}
.GhButton-iconAlign-right {
  flex-direction: row-reverse;
  i {
    margin-left: 8px;
  }
}
.GhButton-fill-blue {
  background-color: #3760ea;
  border-color: #3760ea;
  border-style: solid;
  i,
  p {
    color: #fff;
  }
}
.GhButton-fill-red {
  background-color: #d13532;
  border-color: #d13532;
  border-style: solid;
  i,
  p {
    color: #fff;
  }
}
.GhButton-solid-blue {
  background-color: #ffffff00;
  border-color: #babdcc;
  border-style: solid;
  p {
    color: #121314;
  }
  i {
    color: #d8dae5;
  }
}
.GhButton-solid-red {
  background-color: #ffffff00;
  border-color: #d13532;
  border-style: solid;
  p {
    color: #d13532;
  }
  i {
    color: #d13532;
  }
}
.GhButton-dashed-blue {
  background-color: #ffffff00;
  border-color: #babdcc;
  border-style: dashed;
  p {
    color: #121314;
  }
  i {
    color: #d8dae5;
  }
}
.GhButton-dashed-red {
  background-color: #ffffff00;
  border-color: #d13532;
  border-style: dashed;
  p {
    color: #d13532;
  }
  i {
    color: #d13532;
  }
}

.GhButton-normal {
  .GhButton-fill-blue:hover {
    background-color: #5c87ff;
    border-color: #5c87ff;
  }
  .GhButton-fill-blue:active {
    background-color: #284ac1;
    border-color: #284ac1;
  }

  .GhButton-fill-red:hover {
    background-color: #fa8075;
    border-color: #fa8075;
  }
  .GhButton-fill-red:active {
    background-color: #a92228;
    border-color: #a92228;
  }

  .GhButton-solid-blue:hover {
    background-color: #ffffff00;
    border-color: #5c87ff;
    p {
      color: #5c87ff !important;
    }
    i {
      color: #5c87ff !important;
    }
  }
  .GhButton-solid-blue:active {
    background-color: #f0f6ff;
    border-color: #284ac1;
    p {
      color: #5c87ff !important;
    }
    i {
      color: #284ac1 !important;
    }
  }

  .GhButton-solid-red:hover {
    background-color: #ffffff00;
    border-color: #fa8075;
    p {
      color: #fa8075 !important;
    }
    i {
      color: #fa8075 !important;
    }
  }
  .GhButton-solid-red:active {
    background-color: #fff3f1;
    border-color: #a92228;
    p {
      color: #a92228 !important;
    }
    i {
      color: #a92228 !important;
    }
  }

  .GhButton-dashed-blue:hover {
    background-color: #ffffff00;
    border-color: #5c87ff;
    p {
      color: #5c87ff !important;
    }
    i {
      color: #5c87ff !important;
    }
  }
  .GhButton-dashed-blue:active {
    background-color: #f0f6ff;
    border-color: #284ac1;
    p {
      color: #284ac1 !important;
    }
    i {
      color: #284ac1 !important;
    }
  }

  .GhButton-dashed-red:hover {
    background-color: #ffffff00;
    border-color: #fa8075;
    p {
      color: #fa8075 !important;
    }
    i {
      color: #fa8075 !important;
    }
  }
  .GhButton-dashed-red:active {
    background-color: #fff3f1;
    border-color: #a92228;
    p {
      color: #a92228 !important;
    }
    i {
      color: #a92228 !important;
    }
  }
}
.GhButton-disabled {
  .GhButton {
    cursor: no-drop;
  }
  .GhButton-fill-blue {
    background-color: #b9d0fd;
    border-color: #b9d0fd;
  }
  .GhButton-fill-red {
    background-color: #ffe5e1;
    border-color: #ffe5e1;
  }
  .GhButton-solid-blue {
    border-color: #babdcc;
    p {
      color: #babdcc;
    }
  }
  .GhButton-solid-red {
    border-color: #ffe5e1;
    p {
      color: #ffe5e1;
    }
    i {
      color: #ffe5e1;
    }
  }
  .GhButton-dashed-blue {
    border-color: #babdcc;
    p {
      color: #babdcc;
    }
  }
  .GhButton-dashed-red {
    border-color: #ffe5e1;
    p {
      color: #ffe5e1;
    }
    i {
      color: #ffe5e1;
    }
  }
}
.GhButton-onlyIcon {
  .GhButton {
    i {
      margin-left: 0;
      margin-right: 0;
    }
  }
}
</style>
