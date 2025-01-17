<!-- Created by 337547038 on 2021/6 0027. -->
<template>
  <select-down
    v-bind="$props"
    ref="selectDown"
    before-change=""
    :model-value="showLabel"
    @clear="clearClick"
    @delete="deleteClick"
    @input="inputChange"
    @blur="inputBlur">
    <ul :class="`${prefixCls}-select`">
      <li
        v-for="(item,index) in options"
        v-show="!item._disabled"
        :key="index"
        :class="{'disabled':item.disabled,'active':getActive(item),[item.class]:item.class}"
        :title="item.label"
        @click="itemClick(item,$event)"
        v-html="getItemText(item.label)">
      </li>
    </ul>
    <slot></slot>
  </select-down>
</template>
<script lang="ts">
import {prefixCls} from '../prefix'
import {
  ref,
  defineComponent,
  computed,
  watch,
  inject,
  reactive,
  toRefs,
  provide,
  onMounted
} from 'vue'
import pType from '../util/pType'
import {AnyPropName, FormControlOption} from '../types'
import SelectDown from '../selectDown/index.vue'

export default defineComponent({
  name: `${prefixCls}Select`,
  components: {SelectDown},
  props: {
    modelValue: pType.oneOfType([pType.array(), pType.string(), pType.number()]),
    multiple: pType.bool(), // 是否多选
    multipleLimit: pType.number(0),
    placeholder: pType.string(), // 默认显示的文本
    options: pType.array<FormControlOption>([]), // 下拉选顶
    beforeChange: pType.func(true),
    disabled: pType.bool(),
    filterable: pType.bool(), // 是否可搜索
    async: pType.bool(false), // 异步搜索，filterable=true时有效
    clear: pType.bool(), // 是否可以清空选项
    downHeight: pType.number(0), // 显示下拉最大高度，超出显示滚动条
    downStyle: pType.object(), // 下拉面板样式
    downClass: pType.string(), // 下拉类名
    direction: pType.number(0), // 下拉弹出方向，0自动，1向下，2向上
    appendToBody: pType.bool(),
    width: pType.string(),
    size: pType.string(),
    collapseTags: pType.bool() // 多选时是否将选中值按文字的形式展示
  },
  emits: ['update:modelValue', 'change', 'limitChange', 'searchChange'],
  setup(props, {emit}) {
    const el = ref()
    const selectDown = ref()
    // const optionsList = ref(props.options)
    const state = reactive<AnyPropName>({
      checked: [], // 所有已选择的集合
      keywords: '',
      setFirst: false // 手动选择改变选项时，在watch时不触发setFirstText事件
    })
    watch(() => props.modelValue, () => {
      setFirstText()
    })
    watch(() => props.options, () => {
      setFirstText()
    })
    const inputBlur = (value: string) => {
      // 搜索输入框失焦时，判断输入的值是否符合
      if (!props.filterable) {
        return
      }
      const filter = props.options.filter(item => {
        return item.label === value && !item.disabled
      })
      if (filter.length > 0) {
        // 输入框符合要求，检查下当前项是不是已选择
        let hasItem = false
        state.checked.forEach((item: any) => {
          if (item.label === filter[0].label) {
            hasItem = true
          }
        })
        if (!hasItem) {
          // 没有时添加
          state.checked.push(filter[0])
          emitCom(filter[0])
        }
      }
      // 还原下拉数据
      state.keywords = ''
      setTimeout(() => {
        props.options.forEach(item => {
          delete item._disabled
        })
      }, 500)
    }
    const inputChange = (value: string) => {
      // 默认情况下仅对当前下拉数据进行筛选
      if (!props.filterable) {
        return
      }
      state.keywords = value
      if (!props.async) {
        // 从当前下拉列表筛选
        props.options.forEach(item => {
          item._disabled = item.label.indexOf(value) === -1
        })
      }
      state.setFirst = true
      emit('searchChange', value)
    }
    // 设置初始值
    const setFirstText = () => {
      console.log('setFirstText')
      if (state.setFirst) {
        return
      }
      state.setFirst = false
      if (props.modelValue && props.options && props.options.length > 0) {
        for (let i = 0; i < props.options.length; i++) {
          const item = props.options[i]
          const value = getValueLabel(item)
          if (props.multiple) {
            // 多选
            if (props.modelValue.indexOf(value) !== -1) {
              state.checked.push(item)
            }
          } else {
            // 单选
            state.checked = []
            if (value === props.modelValue) {
              state.checked.push(item) // 没有label时直接取value
              break
            }
          }
        }
      }
    }
    const getActive = (item: FormControlOption) => {
      const val = getValueLabel(item)
      if (props.multiple) {
        return props.modelValue.indexOf(val) !== -1
      } else {
        return val === props.modelValue
      }
    }
    const getValueLabel = (obj: any) => {
      return obj.value === undefined ? obj.label : obj.value
    }
    const itemClick = (item: FormControlOption, evt: MouseEvent) => {
      if (!item.disabled) {
        if (props.beforeChange && !props.beforeChange(item)) {
          slideUp()
          evt.stopPropagation()
          return
        }
        let activeValue = getValueLabel(item) as string
        if (props.multiple) {
          // 多选
          if (props.multipleLimit > 0 && props.multipleLimit <= state.checked.length) {
            emit('limitChange', props.modelValue)
            return false
          }
          let hasItem = -1
          state.checked.forEach((ch: FormControlOption, index: number) => {
            const val = getValueLabel(ch) as string
            if (val === activeValue) {
              // 当前选项已选上，则取消选择
              hasItem = index
            }
          })
          if (hasItem !== -1) {
            state.checked.splice(hasItem, 1)
          } else {
            state.checked.push(item)
          }
          emitCom(item)
        } else {
          // 单选
          state.checked = [item]
          slideUp() // 收起下拉
          emitCom(item)
        }
      }
      evt.stopPropagation()
    }
    const slideUp = () => {
      selectDown.value.slideUp()
    }

    const controlChange: any = inject(`${prefixCls}ControlChange`, '')
    const emitCom = (item?: any, update = true) => {
      state.setFirst = update
      let val: any = []
      state.checked.forEach((obj: FormControlOption) => {
        val.push(getValueLabel(obj))
      })
      if (!props.multiple) {
        val = val.join(',')
      }
      emit('update:modelValue', val)
      controlChange && controlChange(val)
      emit('change', val, item) // 改变事件
    }
    const getItemText = (label: string): string => {
      if (state.keywords && props.filterable) {
        const reg = new RegExp(`${state.keywords}`, 'gi')
        // return label.replace(//gi,)
        return label.replace(reg, '<b>' + '$&' + '</b>')
      } else {
        return label
      }
    }
    // 选中的文本值
    const showLabel = computed(() => {
      return state.checked.map((item: any) => item.label)
    })
    // 清空事件
    const clearClick = () => {
      state.checked = []
      emitCom()
    }
    // 删除选项事件
    const deleteClick = (index: number) => {
      state.checked.splice(index, 1)
      emitCom()
    }
    provide(`${prefixCls}GetChildOption`, (item: FormControlOption) => {
      // options.value.push(item)
      // eslint-disable-next-line vue/no-mutating-props
      props.options.push(item)
    })
    onMounted(() => {
      setFirstText()
    })
    return {
      el,
      selectDown,
      prefixCls,
      ...toRefs(state),
      getActive,
      itemClick,
      getItemText,
      inputChange,
      inputBlur,
      showLabel,
      clearClick,
      deleteClick
    }
  }
})
</script>
