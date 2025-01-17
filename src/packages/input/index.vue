<!-- Created by 337547038 on 2021.6 0015. -->
<template>
  <div
    :class="{[prefixCls+'-form-input']:true,'input-prepend':$slots.prepend,'input-append':$slots.append}"
    :style="{width:width}">
    <div v-if="$slots.prepend" class="prepend">
      <slot name="prepend"></slot>
    </div>
    <input
      v-bind="$attrs"
      ref="inputEl"
      :value="modelValue"
      :type="inputType"
      :class="{'disabled':disabledOk,[prefixCls+'-input-control']:true,'has-prefix':$slots.prefix||prefixIcon,[size]:size}"
      :disabled="disabledOk"
      @input="inputHandler"
      @focus="focusHandler"
      @blur="blurHandler">
    <div v-if="$slots.append" class="append">
      <slot name="append"></slot>
    </div>
    <span v-if="$slots.prefix||prefixIcon" class="prefix-icon">
      <i v-if="prefixIcon" :class="[prefixIcon]"></i>
      <slot name="prefix"></slot>
    </span>
    <span class="suffix-icon">
      <slot name="suffix"></slot>
      <i v-if="suffixIcon" :class="[suffixIcon]"></i>
      <i
        v-if="clear&&modelValue"
        class="icon-close"
        @click.stop="clearValue"></i>
      <i
        v-if="modelValue&&showEye && type==='password'"
        :class="{'icon-eye-close':eyeShow,'icon-eye':!eyeShow}"
        @click.stop="eyeShow=!eyeShow"></i>
    </span>
  </div>
</template>
<script lang="ts">
import {prefixCls} from '../prefix'
import {ref, defineComponent, computed, watch, inject, onMounted} from 'vue'
import pType from '../util/pType'
import {getFormDisabled} from '../util/form'

export default defineComponent({
  name: `${prefixCls}Input`,
  components: {},
  inheritAttrs: false,
  props: {
    modelValue: pType.any(),
    disabled: pType.bool(false),
    type: pType.string('text'),
    clear: pType.bool(false),
    showEye: pType.bool(false), // 密码框显示眼睛，可切换为明文密码
    prefixIcon: pType.string(), // 前缀icon
    suffixIcon: pType.string(), // 后缀icon
    width: pType.string(),
    size: pType.string()
  },
  emits: ['blur', 'focus', 'update:modelValue', 'change'],
  setup(props, {emit}) {
    const inputEl = ref()
    const eyeShow = ref(false)
    const disabledOk = computed(() => {
      return getFormDisabled(props.disabled)
    })
    const inputType = computed(() => {
      if (eyeShow.value) {
        return 'text'
      } else {
        return props.type
      }
    })
    const blurHandler = (e: Event) => {
      emit('blur', e)
      const {value} = e.target as HTMLInputElement
      controlChangeEvent(value, 'blur')
    }
    const focusHandler = (e: Event) => {
      emit('focus', e)
      const {value} = e.target as HTMLInputElement
      controlChangeEvent(value, 'focus')
    }
    const inputHandler = (e: Event) => {
      const {value} = e.target as HTMLInputElement
      emit('update:modelValue', value)
      controlChangeEvent(value)
    }
    const clearValue = () => {
      emit('update:modelValue', '')
      emit('change', '')
    }
    // 通过el.value.focus()设置焦点事件
    const focus = () => {
      inputEl.value.focus()
    }
    const blur = () => {
      inputEl.value.blur()
    }
    watch(() => props.modelValue, (v: any) => {
      controlChangeEvent(v, 'mounted')
    })
    // formItem
    const controlChange: any = inject(`${prefixCls}ControlChange`, '')
    const controlChangeEvent = (val: any, type?: string) => {
      controlChange && controlChange(val, type)
    }
    onMounted(() => {
      controlChangeEvent(props.modelValue, 'mounted')
    })
    return {
      prefixCls,
      inputType,
      blurHandler,
      focusHandler,
      inputHandler,
      clearValue,
      eyeShow,
      focus,
      blur,
      inputEl,
      disabledOk
    }
  }
})
</script>
