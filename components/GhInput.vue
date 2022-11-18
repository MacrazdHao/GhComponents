<template>
  <div
    v-clickoutside="closeSelector"
    :class="[
      'GhInput',
      disabled ? 'GhInput--disabled' : '',
      isFocus && !disabled ? 'GhInput--focus' : '',
      onlySelector ? 'GhInput--selector' : '',
    ]"
    @click="toggleSelector"
  >
    <img v-if="relPrefixIcon" class="prefixIcon" :src="relPrefixIcon" />
    <input
      :ref="`input-${time}`"
      :placeholder="placeholder || ''"
      :disabled="!!disabled"
      :readonly="!!readonly || onlySelector"
      :maxlength="maxLength"
      :type="inputType || 'text'"
      :max="maxNumber"
      :min="minNumber"
      :value="value"
      @focus="handleFocus"
      @blur="handleBlur"
      @input="handleInput"
      @keydown="handleEnter"
    />
    <p v-if="showLength && maxLength" class="length">
      {{ value.length }}/{{ maxLength }}
    </p>
    <img
      v-if="canCopy && type !== 'password'"
      class="copyButton rightButton"
      src="../assets/icon_copy.svg"
      @click="copyValue"
    />
    <img
      v-if="watchPassword && type === 'password'"
      class="watchButton rightButton"
      :src="watchPasswordIcon"
      @click="togglePassword"
    />
    <div v-if="showClear" class="clearButton" @click="clearValue">
      <span> × </span>
    </div>
    <div v-if="hasSuffix" class="suffix">
      <slot name="suffix" />
    </div>
    <img
      v-if="hasSelections && onlySelector"
      :class="[
        'selectorIcon',
        relShowSelectionBox ? 'selectorIcon--show' : 'selectorIcon--hide',
      ]"
      :src="require(`../assets/icon_down${disabled ? '_selected' : ''}.svg`)"
    />
    <!-- selections @scroll="handleScroll" -->
    <div v-if="hasSelections" class="selectionBox">
      <div
        v-show="relShowSelectionBox || animToggle"
        :ref="`selectionBox-${time}`"
        class="selectionBox-inner"
        :style="relSelectionBoxStyle"
        @scroll="handleScroll"
      >
        <template v-if="!relSelectionTips || selectionData">
          <div
            v-for="(item, index) in selectionData"
            :key="`selectionItem-${time}-${index}`"
            :class="['selectionBox-inner-item', `selectionItem-${time}`]"
            @click="handleSelect(index, item)"
          >
            <slot :slot-scope="item" name="selectionItem">
              <p class="selectionBox-inner-item-text">
                {{ item[relSelectionTextKey] }}
              </p>
            </slot>
          </div>
        </template>
        <p
          v-if="relSelectionTips"
          class="selectionBox-inner-tips"
          :style="{ lineHeight: '34px' }"
        >
          {{ relSelectionTips }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * GhInput参数解析
 * @param value[String] 输入框value
 * @param prefixIcon[String(ImageModule)] 左侧Icon，为require的图片资源
 * @param hidePrefixIcon[Boolean] 是否隐藏前置(输入框左侧)Icon
 * @param placeholder[String] 输入框提示语(placeholder)
 * @param disabled[Boolean] 是否禁用输入框
 * @param readonly[Boolean] 是否只读输入框
 * @param maxLength[Number] 最大内容长度
 * @param showLength[Number] 是否显示内容长度
 * @param type[String] 输入框类型(原生input属性)
 * @param watchPassword[Boolean] 是否开启密码显示按钮
 * @param maxNumber[Number] 最大数值(仅type=number时起效)
 * @param minNumber[Number] 最小数值(仅type=number时起效)
 * @param canCopy[Boolean] 是否开启内容复制按钮
 * @param copyTips[Boolean] 复制成功提示
 * @param copyErrTips[String] 复制失败提示
 * @param showClear[Boolean] 是否显示清空内容按钮
 * @param hasSuffix[Boolean] 是否拥有后置(输入框右侧)内容
 * @param hasSelections[Boolean] 是否拥有可选项
 * @param onlySelector[Boolean] 是否仅启动选择功能（该属性为true时会禁用输入功能）
 * @param selectionData[Array({[TextKey||text]: String, ...}]) 选项数据
 * @param selectionTextKey[String] 选项数据中单项的所需展示的文案内容对应Key值
 * @param selectionLoading[Boolean] 选项数据是否正在加载
 * @param selectionError[Boolean] 选项数据是否发生加载错误
 * @param selectionLoadingTips[String] 选项数据加载提示语
 * @param selectionErrorTips[String] 选项数据加载错误提示语
 * @param selectionEmptyTips[String] 选项数据为空提示语
 * @param selectionHideWhenNoKey[Boolean] 是否在value为空时隐藏选择框(onlySelector=true时不生效)
 * @param selectionNumOfPage[Number] 选项数据列表的第一页展示选项数
 */
import Clickoutside from './utilsFromElement/clickoutside'
export default {
  directives: { Clickoutside },
  props: {
    value: {
      type: String,
      default: ''
    },
    prefixIcon: {
      type: String,
      default: null
    },
    hidePrefixIcon: {
      type: Boolean,
      default: false
    },
    placeholder: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    readonly: {
      type: Boolean,
      default: false
    },
    maxLength: {
      type: Number,
      default: Infinity
    },
    type: {
      type: String,
      default: 'text'
    },
    watchPassword: {
      type: Boolean,
      default: false
    },
    maxNumber: {
      type: Number,
      default: 0
    },
    minNumber: {
      type: Number,
      default: 0
    },
    canCopy: {
      type: Boolean,
      default: false
    },
    copyTips: {
      type: String,
      default: ''
    },
    copyErrTips: {
      type: String,
      default: ''
    },
    showLength: {
      type: Boolean,
      default: false
    },
    showClear: {
      type: Boolean,
      default: false
    },
    hasSuffix: {
      type: Boolean,
      default: false
    },
    // 自动建议选择框参数：
    // 包含事件：selectionInitData, selectionLoadMore, select
    hasSelections: {
      type: Boolean,
      default: false
    },
    onlySelector: {
      type: Boolean,
      default: false
    },
    selectionData: {
      type: Object,
      default: null
    },
    selectionTextKey: {
      type: String,
      default: ''
    },
    selectionLoading: {
      type: Boolean,
      default: false
    },
    selectionError: {
      type: Boolean,
      default: false
    },
    selectionLoadingTips: {
      type: String,
      default: ''
    },
    selectionErrorTips: {
      type: String,
      default: ''
    },
    selectionEmptyTips: {
      type: String,
      default: ''
    },
    selectionHideWhenNoKey: {
      type: Boolean,
      default: false
    },
    selectionNumOfPag: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      time: Math.random().toString(36).slice(2),
      isFocus: false,
      showingPassword: false,
      // selections的隐藏标志在animToggle为false时才会生效
      animToggle: true,
      relSelectionBoxStyle: {
        height: 0,
        opacity: 0,
        padding: 0
      }
    }
  },
  computed: {
    // 自动建议选择框中，单项的文字key值：
    relSelectionTextKey() {
      return this.selectionTextKey || 'text'
    },
    relSelectionLoadingTips() {
      return this.selectionLoadingTips || '检索中...'
    },
    relSelectionEmptyTips() {
      return this.selectionErrorTips || '暂无符合条件的结果'
    },
    relSelectionErrorTips() {
      return this.selectionErrorTips || '检索失败，请重试'
    },
    relSelectionTips() {
      if (!this.hasSelections) return false
      if (this.selectionError) return this.relSelectionErrorTips
      if (this.selectionLoading) return this.relSelectionLoadingTips
      if (this.selectionData.length === 0) return this.relSelectionEmptyTips
      return false
    },
    relShowSelectionBox() {
      if (!this.hasSelections || this.disabled || this.readonly) return false
      if (!this.onlySelector && this.selectionHideWhenNoKey && !this.value) {
        return false
      }
      return this.isFocus
    },
    relSelectionNumOfPage() {
      return this.selectionNumOfPage || 5
    },
    selectionBoxStyle() {
      if (!this.hasSelections || !this.relShowSelectionBox) {
        return {
          height: 0,
          opacity: 0,
          overflowY: 'hidden'
        }
      }
      const commonStyle = {
        opacity: 1,
        padding: '4px 0',
        overflowY: 'hidden'
      }
      if (this.selectionData.length === 0 && this.relSelectionTips) {
        return {
          ...commonStyle,
          height: '42px',
          overflowY: 'hidden'
        }
      }
      const singleItems = document.getElementsByClassName(
        `selectionItem-${this.time}`
      )
      const itemNum = Math.min(this.relSelectionNumOfPage, singleItems.length)
      let relHeight = 8
      for (let i = 0; i < itemNum; i++) {
        relHeight += singleItems[i].offsetHeight
      }
      // if (this.relSelectionTips) relHeight += 42
      return {
        ...commonStyle,
        height: `${relHeight}px`,
        overflowY:
          this.relSelectionNumOfPage >= singleItems.length
            ? 'hidden'
            : 'auto'
      }
    },

    relPrefixIcon() {
      if (this.hidePrefixIcon) return false
      if (this.prefixIcon) return this.prefixIcon
      return this.hasSelections && !this.onlySelector
        ? require('../assets/icon_search.svg')
        : false
    },
    watchPasswordIcon() {
      if (this.inputType === 'password') {
        return require('../assets/icon_hide.svg')
      }
      return require('../assets/icon_view.svg')
    },
    inputType() {
      if (this.type === 'password') {
        return this.showingPassword ? 'text' : 'password'
      }
      return this.type
    }
  },
  watch: {
    relShowSelectionBox() {
      this.preShowSelections()
    },
    hasSelections() {
      this.preShowSelections()
    },
    selectionData() {
      if (this.selectionData.length > 0) {
        const item = this.selectionData[0]
        if (item[this.relSelectionTextKey] === this.value) {
          this.$emit('input', item[this.relSelectionTextKey])
          this.$emit('select', 0, item)
        }
      }
      this.preShowSelections()
    },
    relSelectionTips() {
      this.preShowSelections()
    },
    value() {
      // this.$emit('selectionInitData')
    }
  },
  mounted() {
    if (this.hasSelections) {
      this.$emit('selectionInitData')
    }
  },
  methods: {
    toggleSelector() {
      if (this.isFocus) this.closeSelector()
      else if (!this.isFocus) this.showSelector()
    },
    closeSelector() {
      if (this.onlySelector) {
        this.isFocus = false
        this.preCloseSelections()
      }
    },
    showSelector() {
      if (this.onlySelector) {
        this.isFocus = true
        this.preShowSelections()
      }
    },
    handleFocus() {
      if (!this.onlySelector) {
        this.isFocus = true
        this.$emit('focus')
      }
    },
    handleBlur() {
      // 留出时间间隙触发选择选项的绑定事件
      setTimeout(() => {
        this.isFocus = false
        this.$emit('blur')
      }, 100)
    },
    handleInput(e) {
      this.$emit('input', e.target.value)
      if (this.hasSelections) this.$emit('selectionInitData')
    },
    handleEnter(e) {
      if (e.keyCode === 13) {
        this.$emit('enter')
      }
    },
    copyValue() {
      const text = this.$refs[`input-${this.time}`]
      text.disabled = false
      if (navigator.userAgent.match(/(iPhone|iPod|iPad);?/i)) {
        window.getSelection().removeAllRanges()
        const range = document.createRange()
        range.selectNode(text)
        window.getSelection().addRange(range)
        if (document.execCommand('Copy')) {
          alert(this.copyTips || '复制成功')
        } else {
          alert(this.copyErrTips || '复制失败，请尝试手动复制')
          this.showDownloadUrl = true
        }
      } else {
        text.select()
        if (document.execCommand('Copy')) {
          alert(this.copyTips || '复制成功')
        } else {
          alert(this.copyErrTips || '复制失败，请尝试手动复制')
          this.showDownloadUrl = true
        }
        text.blur()
      }
      text.disabled = true
    },
    togglePassword() {
      this.showingPassword = !this.showingPassword
    },
    clearValue() {
      this.$emit('input', '')
      this.$emit('clear')
    },
    preShowSelections() {
      this.animToggle = true
      setTimeout(() => {
        this.relSelectionBoxStyle = this.selectionBoxStyle
      })
    },
    preCloseSelections() {
      this.animToggle = true
      this.relSelectionBoxStyle = this.selectionBoxStyle
      setTimeout(() => {
        this.animToggle = false
      }, 200)
    },
    handleScroll(e) {
      if (this.relSelectionTips) return
      const { scrollTop, offsetHeight } = e.target
      if (offsetHeight - scrollTop <= 42) {
        this.$emit('selectionLoadMore')
      }
    },
    handleSelect(index, item) {
      this.preCloseSelections()
      setTimeout(() => {
        if (this.onlySelector) this.isFocus = false
        this.$emit('input', item[this.relSelectionTextKey])
        this.$emit('select', index, item)
      }, 200)
    }
  }
}
</script>

<style lang="scss" scoped>
.GhInput--selector {
  cursor: pointer;
  input {
    cursor: pointer;
    user-select: none;
  }
}
.GhInput--disabled {
  background-color: #f5f5f5 !important;
  cursor: not-allowed !important;
  input {
    color: #b7b7b7 !important;
    cursor: not-allowed;
  }
  .length {
    color: #b7b7b7 !important;
  }
}
.GhInput--focus {
  border: 1px solid #5c87ff !important;
  box-shadow: 0 0 0 1px #5c88ff50 !important;
  .suffix {
    border-left: 1px solid #5c87ff !important;
  }
}
.GhInput {
  display: flex;
  flex-direction: row;
  align-items: center;
  // height: 100%;
  min-height: 36px;
  max-width: 620px;
  box-sizing: border-box;
  border: 1px solid #d9d9d9;
  transition: all 0.2s;
  border-radius: 2px;
  position: relative;
  background-color: #fff;
  padding: 5px 12px;
  p {
    margin: 0;
    color: #333333;
    font-size: 14px;
    line-height: 20px;
    white-space: nowrap;
  }
  input {
    background: none;
    outline: none;
    border: none;
    margin: 0;
    width: 100%;
    font-size: 14px;
    line-height: 20px;
    height: 20px;
    // margin-left: 29px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  input:disabled {
    color: #999999;
  }
  ::-webkit-input-placeholder {
    /* WebKit, Blink, Edge */
    color: #c7c7c7;
  }
  :-moz-placeholder {
    /* Mozilla Firefox 4 to 18 */
    color: #c7c7c7;
  }
  ::-moz-placeholder {
    /* Mozilla Firefox 19+ */
    color: #c7c7c7;
  }
  :-ms-input-placeholder {
    /* Internet Explorer 10-11 */
    color: #c7c7c7;
  }
  input::-webkit-outer-spin-button,
  input::-webkit-inner-spin-button {
    -webkit-appearance: none;
  }
  input[type="number"] {
    -moz-appearance: textfield;
  }
  .prefixIcon {
    width: 12px;
    height: 12px;
    margin-right: 8px;
  }
  .length {
    font-size: 14px;
    color: #999999;
    line-height: 20px;
  }
  .watchButton {
    width: 14px;
    height: auto;
    cursor: pointer;
    margin-left: 4px;
  }
  .copyButton {
    width: 14px;
    height: auto;
    cursor: pointer;
    margin-left: 4px;
  }
  .clearButton {
    background-color: #999999;
    border-radius: 7px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-left: 4px;
    cursor: pointer;
    span {
      width: 14px !important;
      height: 14px !important;
      color: #fff;
      text-align: center;
      font-size: 12px;
      line-height: 14px;
      user-select: none;
    }
  }
  .suffix {
    margin-left: 4px;
  }
  .selectorIcon {
    margin-left: 8px;
    width: 14px;
    transition: 0.4s;
  }
  .selectorIcon--show {
    transform: rotate(180deg);
  }
  .selectorIcon--hide {
    transform: rotate(360deg);
  }
  .selectionBox {
    position: absolute;
    bottom: 0;
    left: 0;
    transform: translate(0%, 100%);
    z-index: 10;
    width: 100%;
    padding-top: 6px;
    &-inner {
      width: 100%;
      overflow-x: hidden;
      background-color: #fff;
      background: #fff;
      box-shadow: 0px 4px 8px 0px #e0e0e0;
      transition: all 0.2s;
      box-sizing: border-box;
      border-radius: 4px;
      transition: all 0.2s;
      &-tips {
        font-size: 14px;
        color: #d3d3d3;
        // line-height: 34px;
        letter-spacing: normal;
        text-align: center;
      }
      &-item {
        width: 100%;
        min-height: 32px;
        display: flex;
        flex-direction: row;
        align-items: center;
        padding: 6px 12px;
        cursor: pointer;
        transition: all 0.2s;
        ::v-deep p {
          width: 100%;
          font-size: 14px;
          line-height: 20px;
          height: 20px;
          transition: all 0.2s;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
      }
      .selectionBox-inner-item:hover {
        background: #5c87ff;
        box-shadow: 0 0 0 1px #fff;
        ::v-deep p {
          color: #fff;
        }
      }
    }
  }
}
</style>
