<template>
  <div class="mid">
    <div
      class="picker-date-input"
      ref="dateInput"
    >
      <el-input
        v-model="startDate"
        readonly="readonly"
      ></el-input>
      <el-input
        v-model="endDate"
        readonly="readonly"
      ></el-input>
    </div>
    <div
      class="picker-box"
      ref="pickerBox"
      style="display:none;"
      :style="{'height':pickerBoxHeight?pickerBoxHeight:'860%'}"
    >
      <div
        class="date-range"
        v-if="Object.keys(pickerOptions).length!=0&&pickerOptions.shortcuts.length!=0"
      >
        <div
          class="date-range-item"
          v-for="(item,index) in pickerOptions.shortcuts"
          :key="index"
          @click="shortcuts(item.type)"
        >
          {{item.text }}
        </div>
      </div>
      <datepicker
        ref="startTime"
        :side='"startTime"'
        :showLunar="showLunar"
        :calenderOptions='calenderOptions'
        @changeDateRang='changeDateRang'
        @chooseThisDay='chooseThisDay'
        @overThisdays='overThisdays'
      ></datepicker>
      <datepicker
        ref="endTime"
        :side='"endTime"'
        :showLunar="showLunar"
        :calenderOptions='calenderOptions'
        @changeDateRang='changeDateRang'
        @chooseThisDay='chooseThisDay'
        @overThisdays='overThisdays'
      ></datepicker>
    </div>
  </div>
</template>

<script>
import datepicker from './date';

export default {
  components: { datepicker },
  props: ['calenderOptions', 'pickerOptions', 'showLunar', 'pickerBoxHeight'],     // calenderOptions-父组件传过来的限制选择日期, pickerOptions-快捷选项，pickerBoxHeight-选择器盒子高度
  data() {
    return {
      startDate: '',              // 跟day1、day2格式不同
      endDate: '',
      chooseFlag: false,          // 已经选中一个日期
      day1: '',
      day2: '',
    }
  },
  watch: {

  },
  mounted() {
    let that = this;
    let pickerBox = this.$refs.pickerBox;                        // 盒子自身阻止冒泡，其他区域点击隐藏盒子，input的点击显示再另外定义事件
    let dateInput = this.$refs.dateInput;

    document.addEventListener("click", function () {            // 盒子外的元素点击
      if (pickerBox.style.display === "block") {
        that.closePickerBox();
      }
    });
    pickerBox.addEventListener("click", function (event) {      // 盒子自身元素的点击
      event = event || window.event;
      event.stopPropagation();
    });
    dateInput.addEventListener('click', function (event) {      // 输入框的点击
      event = event || window.event;
      that.$emit('closeOtherPicker')                            // 同时隐藏其他日期盒子，然后再设置自身盒子为显示，顺序 ！
      pickerBox.style.display = "block";

      let leftDist = that.$refs.startTime.$refs.main.offsetWidth; // 设置结束日期选择器的left距离
      that.$refs.endTime.$refs.main.style.left = leftDist + 'px'

      that.$emit('visibleChange', true)
      event.stopPropagation();
    })
  },
  methods: {
    closePickerBox() {
      var pickerBox = this.$refs.pickerBox;
      if (pickerBox.style.display !== "none")
        this.$emit('visibleChange', false)
      pickerBox.style.display = "none";
    },
    shortcuts(pickerDays) {

      let end = new Date();
      let start = new Date();

      // 传入的自定义时间,pickerDays若是一个函数，返回数据包括开始时间和结束时间
      if (Object.prototype.toString.call(pickerDays) === '[object Function]') {
        const pickerD = pickerDays();
        start = pickerD[0];
        end = pickerD[1];
      }
      // 若 pickerDays是一个数字，则作为推前的天数
      else if (Object.prototype.toString.call(pickerDays) === '[object Number]') {
        // start = moment(start).subtract(pickerDays, "days").endOf("d").toDate();
        start = new Date(start.getFullYear(), start.getMonth(), start.getDate() - pickerDays)
      }
      else if (pickerDays == 'month') {
        let year = start.getFullYear();
        let mon = start.getMonth() + 1;
        let day = start.getDate() - 1;
        start.setTime(start.getTime() - 3600 * 1000 * 24 * day);
        end = new Date(year, mon, 0);
      }
      else if (pickerDays == 'season') {
        let year = start.getFullYear();
        let mon = start.getMonth() + 1;
        let min = mon;
        let max = mon;
        if (1 <= mon && mon <= 3) {
          min = 1;
          max = 3;
        } else if (4 <= mon && mon <= 6) {
          min = 4;
          max = 6;
        } else if (7 <= mon && mon <= 9) {
          min = 7;
          max = 9;
        } else {
          min = 10;
          max = 12
        }
        let dayStart = new Date(year, min, 0)
        let day = dayStart.getDate() - 1;
        dayStart.setTime(dayStart.getTime() - 3600 * 1000 * 24 * day)
        end = new Date(year, max, 0)
        start = dayStart;
      }
      else if ((pickerDays == 'year')) {
        let year = start.getFullYear();
        end = new Date(year, 12, 0);
        let dayStart = new Date(year - 1, 12, 0)
        dayStart.setTime(dayStart.getTime() + 3600 * 1000 * 24)
        start = dayStart;
      }

      this.startDate = start.getFullYear() + '/' + ('0' + (start.getMonth() + 1)).slice(-2) + '/' + ('0' + start.getDate()).slice(-2);
      this.endDate = end.getFullYear() + '/' + ('0' + (end.getMonth() + 1)).slice(-2) + '/' + ('0' + end.getDate()).slice(-2);
      this.$emit('chooseThisDay', [new Date(this.startDate), new Date(this.endDate)]);    //触发选择父组件事件

      this.$refs.startTime.setDay1(this.startDate);                // 设置日历中日期选择的样式
      this.$refs.startTime.setDay2(this.endDate);
      this.$refs.endTime.setDay1(this.startDate);
      this.$refs.endTime.setDay2(this.endDate);
    },

    changeDateRang(side, state, choosetype) {                     // 选择年份月份时，左边的年份月份选择触发右边的 
      if (side == 'startTime') {
        this.$refs.endTime[choosetype](state);
      }
      else {
        this.$refs.startTime[choosetype](state);
      }
    },
    initDate(selectedDate) {       // 初始化时间
      if (selectedDate.length > 0) {
        this.startDate = selectedDate[0].getFullYear() + '/' + ('0' + (selectedDate[0].getMonth() + 1)).slice(-2) + '/' + ('0' + selectedDate[0].getDate()).slice(-2);
        this.endDate = selectedDate[1].getFullYear() + '/' + ('0' + (selectedDate[1].getMonth() + 1)).slice(-2) + '/' + ('0' + selectedDate[1].getDate()).slice(-2);
        this.$refs.startTime.initDate(selectedDate[0]);
        this.$refs.endTime.initDate(selectedDate[0]);

        this.$refs.startTime.setDay1(this.startDate);               // 设置日历中日期选择的样式
        this.$refs.startTime.setDay2(this.endDate);
        this.$refs.endTime.setDay1(this.startDate);
        this.$refs.endTime.setDay2(this.endDate);
      }
      else {
        this.startDate = '';
        this.endDate = '';
        this.$refs.startTime.setDay1(this.startDate);               // 设置日历中日期选择的样式
        this.$refs.startTime.setDay2(this.endDate);
        this.$refs.endTime.setDay1(this.startDate);
        this.$refs.endTime.setDay2(this.endDate);
      }
    },

    chooseThisDay(time) {
      if (!this.chooseFlag) {                             // 选择的第一个时间,chooseFlag判断是否已经选择一个时间
        this.day1 = time;
        this.chooseFlag = true;
        this.$refs.startTime.setDay1(this.day1);          // 设置日历中日期选择的样式
        this.$refs.startTime.setDay2('');                 // 清空日历选择的样式
        this.$refs.endTime.setDay1(this.day1);
        this.$refs.endTime.setDay2('');
      }
      else if (this.chooseFlag) {                          // 选择的第二个时间，判断哪个是开始时间那个结束时间
        this.day2 = time;
        if (new Date(this.day1.getFullYear(), this.day1.getMonth(), this.day1.getDate()).getTime() < new Date(this.day2.getFullYear(), this.day2.getMonth(), this.day2.getDate())) {
          this.startDate = this.day1.getFullYear() + '/' + ('0' + (this.day1.getMonth() + 1)).slice(-2) + '/' + ('0' + this.day1.getDate()).slice(-2);
          this.endDate = this.day2.getFullYear() + '/' + ('0' + (this.day2.getMonth() + 1)).slice(-2) + '/' + ('0' + this.day2.getDate()).slice(-2);
          this.$refs.startTime.setDay2(this.day2);
          this.$refs.endTime.setDay2(this.day2);
        }
        else {
          this.startDate = this.day2.getFullYear() + '/' + ('0' + (this.day2.getMonth() + 1)).slice(-2) + '/' + ('0' + this.day2.getDate()).slice(-2);
          this.endDate = this.day1.getFullYear() + '/' + ('0' + (this.day1.getMonth() + 1)).slice(-2) + '/' + ('0' + this.day1.getDate()).slice(-2);
          this.$refs.startTime.setDay1(this.day2);                 // 设置日历中日期选择的样式
          this.$refs.startTime.setDay2(this.day1);
          this.$refs.endTime.setDay1(this.day2);
          this.$refs.endTime.setDay2(this.day1);
        }
        this.chooseFlag = false;
        this.closePickerBox();
        // this.$emit('visibleChange', false)
        this.$emit('chooseThisDay', [new Date(this.startDate), new Date(this.endDate)]);    // 选择了两个时间，触发选择父组件事件
      }
    },
    overThisdays(hoverDay, side) {
      if (side == 'startTime') {
        this.$refs.endTime.hoverDay = hoverDay;
      }
      else {
        this.$refs.startTime.hoverDay = hoverDay;
      }
    },
  }
}
</script>