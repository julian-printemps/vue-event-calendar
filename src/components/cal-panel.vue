<template>
  <div class="cal-wrapper">
    <div class="cal-header">
      <div class="l" @click="preMonth"><div class="arrow-left icon">&nbsp</div><span>{{preMonthDisplay}}</span></div>
      <div class="title">{{curYearMonth}}</div>
      <div class="r" @click="nextMonth"><div class="arrow-right icon">&nbsp</div><span>{{nextMonthDisplay}}</span></div>
    </div>
    <section class="cal-shows" v-if="showItemIsSet">
      <h3 class="cal-shows--item">
        上映回数<b>{{ showsItem.nbscreening }}回</b> <strong>残り開催可能数<b>{{ screeningPossibilities(showsItem.nbscreening) }}回</b></strong>
      </h3>
    </section>
    <div class="cal-body">
      <div class="weeks">
        <span
          v-for="(dayName, dayIndex) in i18n[calendar.options.locale].dayNames"
          class="item">
          {{i18n[calendar.options.locale].dayNames[(dayIndex + calendar.options.weekStartOn) % 7]}}
        </span>
      </div>
      <div class="dates" >
        <div v-for="date in dayList" class="item"
          :class="{
            today: date.status ? (today == date.date) : false,
            event: date.status ? (date.isEvent == true) : false,
            nationalday: date.status ? (date.isNationalday == true) : false,
            [calendar.options.className] : (date.date == selectedDay)
          }">
          <p class="date-num"
            @click="handleChangeCurday(date)">
            {{date.status ? date.date.split('/')[2] : '&nbsp'}}</p>
          <span v-if="date.status ? (today == date.date) : false" class="is-today"></span>
          <span v-if="date.status ? (date.isEvent == true) : false" class="is-event"></span>
          <span v-if="date.status ? (date.isNationalday == true) : false" class="is-nationalday"></span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import i18n from '../i18n.js'
import { dateTimeFormatter, isEqualDateStr} from '../tools.js'

const inBrowser = typeof window !== 'undefined'
export default {
  name: 'cal-panel',
  data () {
    return {
      showsItem: {},
      showItemIsSet: false,
      i18n
    }
  },
  props: {
    shows: Array,
    nationaldays: Array,
    events: {
      type: Array,
      required: true
    },
    calendar: {
      type: Object,
      required: true
    },
    selectedDay: {
      type: String,
      required: false
    }
  },
  computed: {
    dayList () {
        let firstDay = new Date(this.calendar.params.curYear, this.calendar.params.curMonth, 1)
        let dayOfWeek = firstDay.getDay()
        // 根据当前日期计算偏移量
        if (this.calendar.options.weekStartOn > dayOfWeek) {
          dayOfWeek = dayOfWeek - this.calendar.options.weekStartOn + 7
        } else if (this.calendar.options.weekStartOn < dayOfWeek) {
          dayOfWeek = dayOfWeek - this.calendar.options.weekStartOn
        }

        let startDate = new Date(firstDay)
        startDate.setDate(firstDay.getDate() - dayOfWeek)

        let item, status, tempArr = [], tempItem
        for (let i = 0 ; i < 42 ; i++) {
            item = new Date(startDate);
            item.setDate(startDate.getDate() + i);

            if (this.calendar.params.curMonth === item.getMonth()) {
              status = 1
            } else {
              status = 0
            }
            tempItem = {
              date: `${item.getFullYear()}/${item.getMonth()+1}/${item.getDate()}`,
              status: status
            }
            this.events.forEach((event) => {
              if (isEqualDateStr(event.date, tempItem.date)) {
                tempItem.isEvent = true
                tempItem.title = event.title
                tempItem.desc = event.desc || ''
              }
            })
            this.nationaldays.forEach((day) => {
              if (isEqualDateStr(day.date, tempItem.date)) {
                tempItem.isNationalday = true
                tempItem.title = day.title
                tempItem.desc = ''
              }
            })
            tempArr.push(tempItem)
        }
        return tempArr
    },
    today () {
      let dateObj = new Date()
      return `${dateObj.getFullYear()}/${dateObj.getMonth()+1}/${dateObj.getDate()}`
    },
    curYearMonth () {
      let tempDate = Date.parse(new Date(`${this.calendar.params.curYear}/${this.calendar.params.curMonth+1}/01`))
      return dateTimeFormatter(tempDate, this.i18n[this.calendar.options.locale].format)
    },
    customColor () {
      return this.calendar.options.color
    },
    preMonthDisplay () {
      let tempDate = Date.parse(new Date(`${this.calendar.params.curYear}/${this.calendar.params.curMonth}/01`))
      return dateTimeFormatter(tempDate, this.i18n[this.calendar.options.locale].monthFormat)
    },
    nextMonthDisplay () {
      let tempDate = Date.parse(new Date(`${this.calendar.params.curYear}/${this.calendar.params.curMonth+2}/01`))
      return dateTimeFormatter(tempDate, this.i18n[this.calendar.options.locale].monthFormat)
    }
  },
  mounted: function () {
    this.setShowsItem()
  },
  methods: {
    setShowsItem () {
      let self = this
      self.showItemIsSet = false
      let tempDate = Date.parse(new Date(`${this.calendar.params.curYear}/${this.calendar.params.curMonth+1}/01`))
      let currentMonth = dateTimeFormatter(tempDate, 'M')
      this.shows.forEach((show) => {
        if (show.month == currentMonth) {
          self.showsItem = show
          self.showItemIsSet = true
        }
      })
    },
    nextMonth () {
      this.$EventCalendar.nextMonth()
      this.$emit('month-changed', this.curYearMonth)
      this.setShowsItem()
    },
    preMonth () {
      this.$EventCalendar.preMonth()
      this.$emit('month-changed', this.curYearMonth)
      this.setShowsItem()
    },
    handleChangeCurday (date) {
      if (date.status) {
        this.$emit('cur-day-changed', date.date)
      }
    },
    screeningPossibilities (nbscreening) {
      return 4 - nbscreening
    }
  }
}
</script>
