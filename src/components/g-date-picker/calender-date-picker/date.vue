<template>
  <div
    class="main"
    ref="main"
  >
    <div class="choose_year">
      <div
        :class="[side,'icon']"
        @click="chooseYears(-1);changeDateRang(side,-1,'chooseYears')"
        v-if="side=='startTime'"
      ><i class="el-icon-d-arrow-left"></i></div>
      <div
        :class="[side,'icon']"
        @click="chooseMonth(-1);changeDateRang(side,-1,'chooseMonth')"
        v-if="side=='startTime'"
      ><i class="el-icon-arrow-left"></i></div>

      <div class="date">{{year}} 年 {{month.toString().padStart(2, '0')}} 月 </div>

      <div
        :class="[side,'icon']"
        @click="chooseMonth(1);changeDateRang(side,1,'chooseMonth')"
        v-if="side=='endTime'"
      ><i class="el-icon-arrow-right"></i></div>
      <div
        :class="[side,'icon']"
        @click="chooseYears(1);changeDateRang(side,1,'chooseYears')"
        v-if="side=='endTime'"
      ><i class="el-icon-d-arrow-right"></i></div>
    </div>

    <div
      class="days_area"
      ref="days"
    >
      <div
        class="day week"
        v-for="week in weeks"
        :key="week"
      >{{week}}</div>
      <div
        class="day"
        @click="chooseThisDay(day.gregorian)"
        @mouseover="overThisdays(day.gregorian)"
        v-for="(day, index) in days"
        :key="index"
        :class="[
        new Date(year, month - 1, day.gregorian).getTime() > calenderOptions?'disabled':'',
        (day1 && day.gregorian && ( new Date(day1).getTime() <= (new Date(year,month-1,day.gregorian).getTime()) && (new Date(year,month-1,day.gregorian).getTime()) <= new Date(hoverDay).getTime()) ) ? 'in-rang':'',
        day1 && day.gregorian && (new Date(year,month-1,day.gregorian).getTime()==new Date(day1).getTime() ) ? 'choose_day' : '', 
        day2 && day.gregorian && (new Date(year,month-1,day.gregorian).getTime()==new Date(day2).getTime() ) ? 'choose_day' : '', 
        (day1 && day2 && day.gregorian && ( new Date(day1).getTime() <= (new Date(year,month-1,day.gregorian).getTime()) && (new Date(year,month-1,day.gregorian).getTime()) <= new Date(day2).getTime()) ) ? 'in-rang':'',

        ]"
      >
        <!-- 难道是 day.gregorian为空的话 设置日期会跟每个日期的gettime相等吗 -->
        <p>{{day.gregorian}}</p>
        <template v-if="showLunar">
          <span v-if="day.festival">{{day.festival}}</span>
          <span v-else-if="day.season">{{day.season}}</span>
          <span v-else>{{day.lunar}}</span>
        </template>

      </div>
    </div>
  </div>

</template>
  
<script>
import calendar from './calendar.js'
export default {
  name: 'calendar',
  props: ['side', 'calenderOptions', 'showLunar'],  // showLunar -- showLunar显不显示农历 ，calenderOptions - 不可选的日期，side-区分开始或结束的一边
  data() {
    return {
      year: 0,
      month: 0,
      today: 0,
      days: [],
      weeks: ['日', '一', '二', '三', '四', '五', '六'],
      day1: '',           // 选中的第一天
      day2: '',           // 选中的第二天
      hoverDay: '',       // 鼠标悬停的日期
    }
  },
  mounted() {
    const now = new Date()
    this.year = now.getFullYear()
    this.month = now.getMonth() + 1
    this.today = now.getDate()
    if (this.side == 'endTime') {                                  // 右边比左边多一个月
      this.month = now.getMonth() + 2
      this.today = 1;
    }
    this.getDays();
  },
  watch: {
  },
  methods: {
    initDate(date) {
      const now = new Date(date)
      this.year = now.getFullYear()
      this.month = now.getMonth() + 1
      this.today = now.getDate()
      if (this.side == 'endTime') {                                  // 右边比左边多一个月
        this.month = now.getMonth() + 2
        this.today = 1;
      }
      this.getDays();
    },
    getDays() {                                                    // 获取当前月份所有公历日期及其农历日期
      this.days = []
      const time = new Date(this.year, this.month, 0)              //月份为0代表一月,这里设置日期的月份不需要减1是因为日期为0，表示日期上个月最后一天，即（2019,4,0）表示3月31
      let nullDays = new Date(this.year, this.month - 1, 1);       // 这个月1号之前空出的星期数
      for (let i = 0; i < nullDays.getDay(); i++) {
        this.days.push({ gregorian: '', lunar: '' })
      }
      for (let i = 1; i <= time.getDate(); i++) {
        let dayObj = calendar.solar2lunar(this.year, this.month, i);
        this.days.push({ gregorian: i, lunar: dayObj.IDayCn == '初一' ? dayObj.IMonthCn : dayObj.IDayCn, season: dayObj.Term, festival: dayObj.festival.join(' ') })
      }
    },
    chooseYears(state) { // 改变年份
      this.year += state
      this.today = 1
      this.getDays()
    },
    chooseMonth(state) { // 改变月份
      this.month += state
      this.today = 1
      if (this.month < 1) {
        this.month = 12
        this.chooseYears(-1)
      } else if (this.month > 12) {
        this.month = 1
        this.chooseYears(1)
      } else {
        this.getDays()
      }
    },
    changeDateRang(side, state, operation) {        // 修改另一边的日期
      this.$emit('changeDateRang', this.side, state, operation);
    },
    chooseThisDay(day) { // 选择某天，当时间选择器用
      if (day > 0) {
        this.today = day
      }
      let time = new Date(this.year, this.month - 1, this.today);
      if (time.getTime() > this.calenderOptions) {     // 可选区域外
      }
      else {
        this.$emit('chooseThisDay', time);
      }
    },
    setDay1(day1) {
      this.day1 = day1;
    },
    setDay2(day2) {
      this.day2 = day2;
    },
    overThisdays(day) {
      if (!this.day2) {                 // day1和day2都选中的时候不显示跟鼠标移动的样式变化
        this.hoverDay = new Date(this.year, this.month - 1, day);
        this.$emit('overThisdays', this.hoverDay, this.side)
      }
    }
  },
}
</script>


