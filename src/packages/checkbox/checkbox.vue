<!--Created by 337547038 on 2021.6.-->
<template>
  <label :class="{[prefixCls+'-checkbox']:true,'checked':checked,'disabled':disabledOk}" @click="changeHandler">
    <span>
      <span class="checkbox-inner icon-check" :class="{'checked':checked}"></span>
      <span v-if="$slots.default" class="checkbox-text"><slot></slot></span>
      <span v-else class="checkbox-text" v-text="label"></span>
    </span>
  </label>
</template>
<script lang="ts">
import {prefixCls} from '../prefix'
import pType from '../util/pType'
import {defineComponent, computed, inject, onMounted, watch} from 'vue'
import {getFormDisabled} from "../util/form";

export default defineComponent({
  name: `${prefixCls}Checkbox`,
  props: {
    disabled: pType.bool(),
    label: pType.string(),
    modelValue: [String, Boolean, Array],
    value: pType.string(),
    beforeChange: pType.func(true)
  },
  emits: ['change', 'update:modelValue'],
  setup(props, {emit}) {
    const disabledOk = computed(() => {
      return getFormDisabled(props.disabled)
    })
    const checked = computed(() => {
      // value为真时，当v-model=value时为选中状态
      // 否则，当v-model=true时为选中状态
      let bool
      if (props.value) {
        if (typeof props.modelValue === 'object') {
          bool = props.modelValue.indexOf(props.value) !== -1
        } else {
          bool = props.value === props.modelValue
        }
      } else {
        bool = !!props.modelValue
      }
      return bool
    })
    // formItem
    const controlChange: any = inject(`${prefixCls}ControlChange`, '')
    const changeHandler = () => {
      if (props.beforeChange && !props.beforeChange()) {
        return
      }
      // 点击后只有选中状态
      if (disabledOk.value) {
        return
      }
      let val
      if (checked.value) {
        // 当前勾选状态，
        if (typeof props.modelValue === 'object') {
          // 删除当前项
          val = props.modelValue.filter(item => {
            return item !== props.value
          })
        } else {
          val = false
        }
      } else {
        if (props.value) {
          if (typeof props.modelValue === 'object') {
            val = props.modelValue
            val.push(props.value)
          } else {
            val = props.value
          }
        } else {
          val = !props.modelValue
        }
      }
      emit('change', val)
      emit('update:modelValue', val)
      // controlChange && controlChange(val)
      controlChangeEvent(val)
    }
    watch(() => props.modelValue, (v: any) => {
      controlChange && controlChange(props.modelValue, 'mounted')
      controlChangeEvent(v, 'mounted')
    })
    onMounted(() => {
      // controlChange && controlChange(props.modelValue, 'mounted')
      controlChangeEvent(props.modelValue, 'mounted')
    })
    const controlChangeEvent = (value: any, type?: string) => {
      controlChange && controlChange(value, type)
    }
    return {
      prefixCls,
      checked,
      changeHandler,
      disabledOk
    }
  }
})
</script>
