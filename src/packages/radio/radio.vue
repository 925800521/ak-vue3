<!-- Created by 337547038 on 2018/9/7 0007. -->
<template>
  <label
    :class="{
      'disabled':disabledOk,
      'checked':checked,[prefixCls+'-radio']:true}"
    @click="changeHandler">
    <span>
      <span class="radio-inner"></span>
      <span v-if="$slots.default" class="radio-text">
        <slot></slot>
      </span>
      <span v-if="label" class="radio-text" v-html="label"></span>
    </span>
  </label>
</template>
<script lang="ts">
import {prefixCls} from '../prefix'
import pType from '../util/pType'
import {inject, computed, defineComponent, onMounted, watch} from 'vue'
import {getFormDisabled} from '../util/form'

export default defineComponent({
  name: `${prefixCls}Radio`,
  props: {
    disabled: pType.bool(false),
    value: [String, Boolean, Number],
    modelValue: [String, Boolean, Number],
    label: pType.string()
  },
  emits: ['change', 'update:modelValue'],
  setup(props, {emit}) {
    const disabledOk = computed(() => {
      return getFormDisabled(props.disabled)
    })
    const checked = computed(() => {
      // 存在value时，当v-model=value时为选中状态
      // 不存在value时，当v-model=true时为选中状态
      let bool
      if (props.value) {
        bool = props.value === props.modelValue
      } else {
        bool = !!props.modelValue
      }
      return bool
    })
    // formItem
    const controlChange: any = inject(`${prefixCls}ControlChange`, '')
    const changeHandler = () => {
      // 点击后只有选中状态
      if (disabledOk.value) {
        return
      }
      const val = props.value || true
      emit('change', val)
      emit('update:modelValue', val)
      controlChange && controlChange(val)
    }
    watch(() => props.modelValue, (v: any) => {
      controlChange && controlChange(v, 'mounted')
    })
    onMounted(() => {
      controlChange && controlChange(props.modelValue, 'mounted')
    })
    return {
      prefixCls,
      checked,
      changeHandler,
      disabledOk
    }
  }
})
</script>
