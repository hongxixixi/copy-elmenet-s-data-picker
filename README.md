        ref="curDatePicker1"
        v-model="createDate111"
        :showLunar="showLunar111"
        :pickerOptions="pickerOptions"
        :calenderOptions="calenderOptions111"
        @datedata="handleDate"
        @visibleChange="visibleChange1"
        @closeOtherPicker="closeOtherPicker"


        closeOtherPicker() {
          this.$refs.curDatePicker.closeSelf();
          this.$refs.curDatePicker1.closeSelf();
          this.$refs.curDatePicker2.closeSelf();
        },

        this.$refs.curDatePicker.clearvalue(); // 清空数据
        // v-model--选中的日期值
        // pickerOptions-快捷选项 对象 里面的type可以是字符串也可以是函数，还可以是数字
        <!--  pickerDays为一个数字，则直接作为推前的天数 ； pickerDays若是一个函数，返回数据包括开始时间和结束时间，以此时间作为选择区间 ；
        若为 month、season、year字符串、则分别为按月份、季节、年份计算   -->
        // pickerBoxHeight 设置盒子的高度，可以为百分数和数值+'px'，默认是860%；百分数相对的数值是输入框input的高
        // calenderOptions-父组件传过来的限制选择日期
        // showLunar 是否显示农历
        // datedata - 选择日期触发
        // visibleChange - 日期下拉选择收起与展示 ，参数为true或者false
        // closeOtherPicker 页面中多于一个日期选择器时，需要再绑定closeOtherPicker事件，每次触发在内部调用每个!日期选择组件的closeSelf()事件
