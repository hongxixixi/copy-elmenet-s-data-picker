<template>
  <div
    class="g-date-picker"
    id="group-date-picker"
  >
    <div class="calender-date-picker">
      <calenderDatePicker
        ref="date"
        :calenderOptions='calenderOptions'
        :showLunar='showLunar'
        :pickerOptions="pickerOptions"
        :pickerBoxHeight="pickerBoxHeight"
        @chooseThisDay="valueChanged($event); "
        @closeOtherPicker='closeOtherPicker'
        @visibleChange='visibleChange'
      ></calenderDatePicker>
    </div>
  </div>
</template>
<script>

import Vue from 'vue';
import calenderDatePicker from '../calender-date-picker/datepicker';

export default {
  name: "g-date-picker",
  components: {
    calenderDatePicker
  },
  props: {
    value: {      //  选中的时间范围
      type: Array,
      default: () => {
        return []
      }
    },
    pickerOptions: {  //快捷方式时间段
      type: Object,
      default: () => {
        return { shortcuts: [] }
      }
    },
    calenderOptions: {  // 限制选择的时间

    },
    showLunar: {
      type: Boolean,
      default: () => {
        return true
      }
    },
    pickerBoxHeight: {}
  },
  data() {
    return {
      startTime: "",
      endTime: "",
    }
  },
  mounted() {
    let vm = this;

    this.$nextTick(() => {
      //  设置默认选中时间
      if (this.value && this.value.length) {
        this.$refs.date.chooseThisDay(this.value[0])
        this.$refs.date.chooseThisDay(this.value[1])
      }
    })
  },
  methods: {
    visibleChange(visibParam) {
      this.$emit('visibleChange', visibParam)
    },
    // 隐藏其他盒子
    closeOtherPicker() {
      this.$emit('closeOtherPicker')
    },
    // 隐藏自身日期盒子
    closeSelf() {
      this.$refs.date.closePickerBox();
    },
    valueChanged(e) {
      // 选择日期之后改变value, 父组件通过v-model传过来的的value属性需要通过input事件触发改变
      this.$emit('input', [new Date(e[0]), new Date(e[1])])
      this.$emit("handleDate", [new Date(e[0]), new Date(e[1])])
    },
    clearvalue() {
      this.$emit('input', [])
      this.$emit("handleDate", [])
      this.$refs.date.initDate([])   // 清空显示日期的input框
    },
  },
  watch: {
    value() {
    }
  },
  destoryed() {
  }
}
</script>
