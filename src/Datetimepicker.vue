<template>
    <div class="datepicker">
        <input class="form-control datepicker-input" :class="{'with-reset-button': clearButton}" type="text" :placeholder="placeholder" readonly :style="{width:width}" @click="inputClick" v-model="p_value" />
        <span v-if="clearButton && value" type="button" class="close" @click="p_value = ''">
            <span>&times;</span>
        </span>
        <div class="datepicker-popup" v-show="displayDayView">
            <div class="datepicker-inner">
                <div class="datepicker-body">
                    <div class="datepicker-ctrl">
                        <span class="datepicker-preBtn" aria-hidden="true" @click="preNextMonthClick(0)">←</span>
                        <span class="datepicker-nextBtn" aria-hidden="true" @click="preNextMonthClick(1)">→</span>
                        <p @click="switchMonthView">{{stringifyDayHeader(currDate)}}</p>
                    </div>
                    <div class="datepicker-weekRange">
                        <span v-for="w in text.daysOfWeek">{{w}}</span>
                    </div>
                    <div class="datepicker-dateRange">
                        <span v-for="d in dateRange" :class="d.sclass" @click="daySelect(d.date,d.sclass)">{{d.text}}</span>
                    </div>
                </div>
            </div>
        </div>
        <div class="datepicker-popup" v-show="displayMonthView">
            <div class="datepicker-inner">
                <div class="datepicker-body">
                    <div class="datepicker-ctrl">
                        <span class="datepicker-preBtn" aria-hidden="true" @click="preNextYearClick(0)">←</span>
                        <span class="datepicker-nextBtn" aria-hidden="true" @click="preNextYearClick(1)">→</span>
                        <p @click="switchDecadeView">{{stringifyYearHeader(currDate)}}</p>
                    </div>
                    <div class="datepicker-monthRange">
                        <template v-for="(m,index) in text.months">
                            <span :class="{'datepicker-dateRange-item-active':
                  (text.months[parse(p_value).getMonth()]  === m) &&
                  currDate.getFullYear() === parse(p_value).getFullYear()}" @click="monthSelect(index)">{{m.substr(0,3)}}</span>
                        </template>
                    </div>
                </div>
            </div>
        </div>
        <div class="datepicker-popup" v-show="displayYearView">
            <div class="datepicker-inner">
                <div class="datepicker-body">
                    <div class="datepicker-ctrl">
                        <span class="datepicker-preBtn" aria-hidden="true" @click="preNextDecadeClick(0)">←</span>
                        <span class="datepicker-nextBtn" aria-hidden="true" @click="preNextDecadeClick(1)">→</span>
                        <p>{{stringifyDecadeHeader(currDate)}}</p>
                    </div>
                    <div class="datepicker-monthRange decadeRange">
                        <template v-for="decade in decadeRange">
                            <span :class="{'datepicker-dateRange-item-active':
                  parse(this.p_value).getFullYear() === decade.text}" @click.stop="yearSelect(decade.text)">{{decade.text}}</span>
                        </template>
                    </div>
                </div>
            </div>
        </div>
        <div class="datepicker-popup" v-show="displayTimeView" v-if="hasTime">
            <div class="datepicker-inner">
                <div class="datepicker-body">
                    <div class="datepicker-ctrl">
                        <span class="datepicker-preBtn" aria-hidden="true" @click="backToDayView()">←</span>
                        {{hour}}:{{minute}}
                    </div>
                    <div class="datepicker-timeRange">
                        <span class="d">
              <ul>
                <li v-for="item in hours">
                  <input type="radio" :id="'h_'+item+'_'+currTime" :name="'h_radio_'+currTime" :value="item" v-model="hour">
                  <label :for="'h_'+item+'_'+currTime">{{item}}</label>
                </li>
              </ul>
            </span>
                        <span class="d">
              <ul>
                <li v-for="item in minutes">
                  <input type="radio" :id="'m_'+item+'_'+currTime" :name="'m_radio_'+currTime" :value="item" v-model="minute">
                  <label :for="'m_'+item+'_'+currTime">{{item}}</label>
                </li>
              </ul>
            </span>
                    </div>
                    <div class="btn-footer">
                        <button :class="btnClass" :style="btnStyle" type="button" @click="setTime">{{btnText}}</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import {
    translations
} from './utils/utils.js'
import $ from './utils/NodeList.js'

export default {
    props: {
        value: {
            type: String
        },
        format: {
            default: 'mm/dd/yyyy'
        },
        disabledDaysOfWeek: {
            type: Array,
            default () {
                return []
            }
        },
        width: {
            type: String,
            default: '160px'
        },
        clearButton: {
            type: Boolean,
            default: false
        },
        lang: {
            type: String,
            default: navigator.language
        },
        placeholder: {
            type: String
        },
        btnText: {
            type: String,
            default: '确定'
        },
        btnClass: {
            type: String,
            default: ''
        },
        btnStyle: {
            type: Object,
            default: null
        },
        hasTime:{
            type:Boolean,
            default: false
        },
    },
    mounted() {
        this._blur = (e) => {
            if (this.$el !== null && !this.$el.contains(e.target)) this.close()
        }
        this.currDate = this.parse(this.p_value) || this.parse(new Date())
        $(window).on('click', this._blur)

    },
    beforeDestroy() {
        $(window).off('click', this._blur)
    },
    data() {
        return {
            p_value: this.value, //属性value的副本
            currDate: new Date(),
            currTime: new Date().getTime(),
            dateRange: [],
            decadeRange: [],
            displayDayView: false,
            displayMonthView: false,
            displayYearView: false,
            displayTimeView: false,
            hour: '00',
            minute: '00',
            hours: ['00', '01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23'],
            minutes: ['00', '01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24', '25', '26', '27', '28', '29', '30', '31', '32', '33', '34', '35', '36', '37', '38', '39', '40', '41', '42', '43', '44', '45', '46', '47', '48', '49', '50', '51', '52', '53', '54', '55', '56', '57', '58', '59']
        }
    },
    watch: {
        currDate() {
            this.getDateRange()
        },
        p_value(val) {
            this.$emit("on-data-change", val)
        }
    },
    computed: {
        text() {
            return translations(this.lang)
        }
    },
    methods: {
        close() {
            if (!this.displayTimeView) {
                this.displayDayView = this.displayMonthView = this.displayYearView = this.displayTimeView = false
            }
        },
        inputClick() {
            this.currDate = this.parse(this.value) || this.parse(new Date())
            if (this.displayMonthView || this.displayYearView) {
                this.displayDayView = false
            } else {
                this.displayDayView = !this.displayDayView
            }
        },
        preNextDecadeClick(flag) {
            const year = this.currDate.getFullYear()
            const months = this.currDate.getMonth()
            const date = this.currDate.getDate()

            if (flag === 0) {
                this.currDate = new Date(year - 10, months, date)
            } else {
                this.currDate = new Date(year + 10, months, date)
            }
        },
        preNextMonthClick(flag) {
            const year = this.currDate.getFullYear()
            const month = this.currDate.getMonth()
            const date = this.currDate.getDate()

            if (flag === 0) {
                const preMonth = this.getYearMonth(year, month - 1)
                this.currDate = new Date(preMonth.year, preMonth.month, date)
            } else {
                const nextMonth = this.getYearMonth(year, month + 1)
                this.currDate = new Date(nextMonth.year, nextMonth.month, date)
            }
        },
        preNextYearClick(flag) {
            const year = this.currDate.getFullYear()
            const months = this.currDate.getMonth()
            const date = this.currDate.getDate()

            if (flag === 0) {
                this.currDate = new Date(year - 1, months, date)
            } else {
                this.currDate = new Date(year + 1, months, date)
            }
        },
        backToDayView(){
          this.displayYearView = false
          this.displayMonthView = false
          this.displayDayView = true
          this.displayTimeView = false
        },
        yearSelect(year) {
            this.displayYearView = false
            this.displayMonthView = true
            this.currDate = new Date(year, this.currDate.getMonth(), this.currDate.getDate())
        },
        daySelect(date, klass) {
            if (klass === 'datepicker-item-gray'||klass ==='datepicker-item-disable') {
                return false
            } else {
                this.currDate = date
                this.p_value = this.stringify(this.currDate)
                this.displayDayView = false
                this.displayTimeView = true
            }
        },
        switchMonthView() {
            this.displayDayView = false
            this.displayMonthView = true
        },
        switchDecadeView() {
            this.displayMonthView = false
            this.displayYearView = true
        },
        monthSelect(index) {
            this.displayMonthView = false
            this.displayDayView = true
            this.currDate = new Date(this.currDate.getFullYear(), index, this.currDate.getDate())
        },
        getYearMonth(year, month) {
            if (month > 11) {
                year++
                month = 0
            } else if (month < 0) {
                year--
                month = 11
            }
            return {
                year: year,
                month: month
            }
        },
        stringifyDecadeHeader(date) {
            const yearStr = date.getFullYear().toString()
            const firstYearOfDecade = yearStr.substring(0, yearStr.length - 1) + 0
            const lastYearOfDecade = parseInt(firstYearOfDecade, 10) + 10
            return firstYearOfDecade + '-' + lastYearOfDecade
        },
        stringifyDayHeader(date) {
            return this.text.months[date.getMonth()] + ' ' + date.getFullYear()
        },
        parseMonth(date) {
            return this.text.months[date.getMonth()]
        },
        stringifyYearHeader(date) {
            return date.getFullYear()
        },
        stringify(date, format = this.format) {
            if (!date) date = this.parse()
            if (!date) return ''
            const year = date.getFullYear()
            const month = date.getMonth() + 1
            const day = date.getDate()
            const monthName = this.parseMonth(date)

            return format
                .replace(/yyyy/g, year)
                .replace(/mmmm/g, monthName)
                .replace(/mmm/g, monthName.substring(0, 3))
                .replace(/mm/g, ('0' + month).slice(-2))
                .replace(/dd/g, ('0' + day).slice(-2))
                .replace(/yy/g, year)
                .replace(/M(?!a)/g, month)
                .replace(/d/g, day)
        },
        parse(str = this.p_value) {
            let date
            if (str.length === 10 && (this.format === 'dd-mm-yyyy' || this.format === 'dd/mm/yyyy')) {
                date = new Date(str.substring(6, 10), str.substring(3, 5) - 1, str.substring(0, 2))
            } else {
                date = new Date(str)
            }
            return isNaN(date.getFullYear()) ? new Date() : date
        },
        getDayCount(year, month) {
            const dict = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
            if (month === 1) {
                if ((year % 400 === 0) || (year % 4 === 0 && year % 100 !== 0)) {
                    return 29
                }
            }
            return dict[month]
        },
        getDateRange() {
            this.dateRange = []
            this.decadeRange = []
            const time = {
                year: this.currDate.getFullYear(),
                month: this.currDate.getMonth(),
                day: this.currDate.getDate()
            }
            const yearStr = time.year.toString()
            const firstYearOfDecade = (yearStr.substring(0, yearStr.length - 1) + 0) - 1
            for (let i = 0; i < 12; i++) {
                this.decadeRange.push({
                    text: firstYearOfDecade + i
                })
            }

            const currMonthFirstDay = new Date(time.year, time.month, 1)
            let firstDayWeek = currMonthFirstDay.getDay() + 1
            if (firstDayWeek === 0) {
                firstDayWeek = 7
            }
            const dayCount = this.getDayCount(time.year, time.month)
            if (firstDayWeek > 1) {
                const preMonth = this.getYearMonth(time.year, time.month - 1)
                const prevMonthDayCount = this.getDayCount(preMonth.year, preMonth.month)
                for (let i = 1; i < firstDayWeek; i++) {
                    const dayText = prevMonthDayCount - firstDayWeek + i + 1
                    this.dateRange.push({
                        text: dayText,
                        date: new Date(preMonth.year, preMonth.month, dayText),
                        sclass: 'datepicker-item-gray'
                    })
                }
            }
            for (let i = 1; i <= dayCount; i++) {
                const date = new Date(time.year, time.month, i)
                const week = date.getDay()
                let sclass = '';
                this.disabledDaysOfWeek.forEach((el) => {
                    if (week === parseInt(el, 10)) sclass = 'datepicker-item-disable'
                })
                if (i === time.day) {
                    if (this.p_value) {
                        const valueDate = this.parse(this.p_value)
                        if (valueDate) {
                            if (valueDate.getFullYear() === time.year && valueDate.getMonth() === time.month) {
                                sclass = 'datepicker-dateRange-item-active'
                            }
                        }
                    }
                }
                //开始时间 和结束时间 大小限制

                this.dateRange.push({
                    text: i,
                    date: date,
                    sclass: sclass
                })
            }

            if (this.dateRange.length < 42) {
                const nextMonthNeed = 42 - this.dateRange.length
                const nextMonth = this.getYearMonth(time.year, time.month + 1)

                for (let i = 1; i <= nextMonthNeed; i++) {
                    this.dateRange.push({
                        text: i,
                        date: new Date(nextMonth.year, nextMonth.month, i),
                        sclass: 'datepicker-item-gray'
                    })
                }
            }
        },
        setTime() {
            this.p_value = this.stringify(this.currDate) + " " + this.hour + ":" + this.minute
            this.displayTimeView = false
        }
    }
}
</script>
<style>
.datepicker {
    position: relative;
    display: inline-block;
}

input.datepicker-input.with-reset-button {
    padding-right: 25px;
}

.datepicker > span.close {
    position: absolute;
    top: 0;
    right: 0;
    outline: none;
    z-index: 2;
    display: block;
    width: 20px;
    height: 100%;
    text-align: center;
    cursor: default;
}

.datepicker > button.close:focus {
    opacity: .2;
}

.datepicker-popup {
    position: absolute;
    border: 1px solid #ccc;
    border-radius: 5px;
    background: #fff;
    margin-top: 2px;
    z-index: 1000;
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.175);
}

.datepicker-inner {
    width: 218px;
}

.datepicker-body {
    padding: 10px 10px;
}

.datepicker-ctrl p,
.datepicker-ctrl span,
.datepicker-body span {
    display: inline-block;
    width: 28px;
    line-height: 28px;
    height: 28px;
}

.datepicker-ctrl p {
    width: 65%;
    margin: 0;
}

.datepicker-ctrl span {
    position: absolute;
}

.datepicker-body span {
    text-align: center;
}

.datepicker-monthRange span {
    width: 48px;
    height: 50px;
    line-height: 45px;
}

.datepicker-item-disable {
    background-color: white!important;
    cursor: not-allowed!important;
}

.decadeRange span:first-child,
.decadeRange span:last-child,
.datepicker-item-disable,
.datepicker-item-gray {
    color: #999;
}

.datepicker-dateRange-item-active:hover,
.datepicker-dateRange-item-active {
    background: rgb(50, 118, 177)!important;
    color: white!important;
}

.datepicker-monthRange {
    margin-top: 10px
}

.datepicker-monthRange span,
.datepicker-ctrl span,
.datepicker-ctrl p,
.datepicker-dateRange span {
    cursor: pointer;
}

.datepicker-monthRange span:hover,
.datepicker-ctrl p:hover,
.datepicker-ctrl i:hover,
.datepicker-dateRange span:hover,
.datepicker-dateRange-item-hover {
    background-color: #eeeeee;
}

.datepicker-weekRange span {
    font-weight: bold;
}

.datepicker-label {
    background-color: #f8f8f8;
    font-weight: 700;
    padding: 7px 0;
    text-align: center;
}

.datepicker-ctrl {
    position: relative;
    height: 30px;
    line-height: 30px;
    font-weight: bold;
    text-align: center;
}

.month-btn {
    font-weight: bold;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

.datepicker-preBtn {
    left: 2px;
}

.datepicker-nextBtn {
    right: 2px;
}

.datepicker-preBtn:hover,
.datepicker-nextBtn:hover {
    background-color: #eeeeee;
}

.datepicker-timeRange {
    height: 164px;
    width: 200px;
}

.datepicker-timeRange .d {
    width: 97px;
    overflow-y: auto;
    height: 100%;
}

.datepicker-timeRange .d ul li {
    cursor: pointer;
}

.datepicker-timeRange .d ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.datepicker-timeRange .d ul li:hover {
    background-color: #eeeeee;
}

.datepicker-timeRange .d ul li input {
    display: none;
}

.datepicker-timeRange .d ul li label {
    width: 100%;
    display: inline-block;
}

.datepicker-timeRange .d ul li input:checked+label {
    background: rgb(50, 118, 177)!important;
    color: white!important;
}

.datepicker-timeRange .d::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
    background-color: #F5F5F5;
}

.datepicker-timeRange .d::-webkit-scrollbar {
    width: 6px;
    background-color: #F5F5F5;
}

.datepicker-timeRange .d::-webkit-scrollbar-thumb {
    background-color: lightblue;
}

.btn-footer {
    margin: 5px 0px;
}
</style>
