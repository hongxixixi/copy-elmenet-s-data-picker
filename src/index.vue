<template>
  <div class="ds-lunar-datepicker">
    <div class="item-content">
      <g-date-picker
        ref="curDatePicker"
        v-model="selectedDate"
        :showLunar="showLunar"
        :pickerOptions="pickerOptions"
        pickerBoxHeight="340px"
        :calenderOptions="calenderOptions"
        @handleDate="handleDate"
        @visibleChange="visibleChange"
        @closeOtherPicker="closeOtherPicker"
      ></g-date-picker>
    </div>
    <div class="item-content">
      <g-date-picker
        ref="curDatePickerSecond"
        v-model="selectedDateSecond"
        pickerBoxHeight="800%"
        :pickerOptions="pickerOptionsSecond"
        @handleDate="handleDateSecond"
        @visibleChange="visibleChangeSecond"
        @closeOtherPicker="closeOtherPicker"
      ></g-date-picker>
    </div>
  </div>
</template>

<script>
import gDatePicker from './components/g-date-picker/src/index'
import './index.less'
import moment from 'moment'

export default {
  name: "ds-lunar-datepicker",
  components: {
    gDatePicker
  },
  data() {
    return {
      pickerBoxHeight: '280px',
      showLunar: true,// 显示农历
      calenderOptions: new Date(new Date().getTime() + 8 * 86400000), // 自定义旧历限制选择的时间
      selectedDate: [new Date(2018, 6, 4), new Date()],
      pickerOptions: {
        shortcuts: [{
          text: "一周",
          type: () => {
            let end = new Date();
            let start = new Date();
            start = moment(start).add(-1, "w").endOf("d").toDate();
            end = moment(end).startOf("d").toDate();
            return [start, end]
          },
        }, {
          text: "一年",
          type: () => {
            let end = new Date();
            let start = new Date();
            start = moment(start).add(-1, "y").endOf("d").toDate();
            end = moment(end).startOf("d").toDate();
            return [start, end]
          },
        },
        {
          text: "月",
          type: "month"
        },
        {
          text: "季",
          type: "season"
        },
        {
          text: "年",
          type: "year"
        },
        {
          text: "10天",
          type: 10
        },
        {
          text: "18天",
          type: 18
        },
        {
          text: "20天",
          type: 20
        },
        ]
      },



      selectedDateSecond: [],
      // showLunarSecond: false,
      // calenderOptionsSecond: new Date(new Date().getTime() + 86400000),
      selectedDateSecond: [new Date(2019, 6, 4), new Date()],
      pickerOptionsSecond: {
        shortcuts: [
          // { text: '15天', type: 15 },
          // { text: '45天', type: 45 },
        ]
      }
    }
  },
  methods: {
    handleDateSecond() {
    },
    visibleChangeSecond(visibParam) {
    },

    visibleChange(visibParam) {
      // console.log(visibParam)
    },
    handleDate(data) {
      // console.log(data + 'handle')
    },
    closeOtherPicker() {
      this.$refs.curDatePicker.closeSelf();
      this.$refs.curDatePickerSecond.closeSelf();
    },
  },
  watch: {
    selectedDate() {
      // console.log(this.selectedDate + 'value改变')
    }
  },
  mounted() {
    setTimeout(() => {
      // this.$refs.curDatePicker.clearvalue();
    }, 4000)
  }
}
</script>

<style lang="less" scoped>
.ds-lunar-datepicker {
  .item-content {
    position: relative;
    font-size: 14px;
    width: 480px;
    height: 32px;
    float: left;
    margin-left: 8rem;
  }
}
</style>
