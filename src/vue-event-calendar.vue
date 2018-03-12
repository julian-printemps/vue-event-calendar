<template>
  <div class="__vev_calendar-wrapper">
    <cal-panel
      :shows="shows" :events="events" :nationaldays="nationaldays" :disableddays="disableddays"
      :calendar="calendarOptions"
      :selectedDay='selectedDayEvents.date'
      @cur-day-changed="handleChangeCurDay"
      @month-changed="handleMonthChanged">
    </cal-panel>
    <cal-events
      :dayEvents="selectedDayEvents"
      :locale="calendarOptions.options.locale"
      :color="calendarOptions.options.color">
      <slot :showEvents="selectedDayEvents.events"></slot>
    </cal-events>
  </div>
</template>
<script>
import { isEqualDateStr} from './tools.js'

import calEvents from './components/cal-events.vue'
import calPanel from './components/cal-panel.vue'

const inBrowser = typeof window !== 'undefined'
export default {
  name: 'vue-event-calendar',
  components: {
    'cal-events': calEvents,
    'cal-panel': calPanel
  },
  data () {
    return {
      selectedDayEvents: {
        date: 'all',
        events: this.events || []  //default show all event
      },

      // shows: [
      //   {
      //     nbscreening: 4,
      //     month: 8
      //   },
      //   {
      //     nbscreening: 2,
      //     month: 9
      //   },
      //   {
      //     nbscreening: 1,
      //     month: 10
      //   },
      //   {
      //     nbscreening: 3,
      //     month: 11
      //   },
      //   {
      //     nbscreening: 0,
      //     month: 12
      //   }
      // ],
      // nationaldays: [
      //   {
      //     date: '2017/09/12',
      //     title: 'National day'
      //   },
      //   {
      //     date: '2017/10/10',
      //     title: 'Bar'
      //   }
      // ],
    }
  },
  props: {
    events: {
      type: Array,
      required: true,
      default: [],
      validator (events) {
        let validate = true
        events.forEach((event, index) => {
          if (!event.date) {
            console.error('Vue-Event-Calendar-Error:' + 'Prop events Wrong at index ' + index)
            validate = false
          }
        })
        return validate
      }
    // }
    },
    shows: Array,
    nationaldays: Array,
    disableddays: Array
  },
  computed: {
    calendarOptions () {
      let dateObj = new Date()
      if (inBrowser) {
          return window.VueCalendarBarEventBus.CALENDAR_EVENTS_DATA
      } else {
        return {
          options: {
            locale: 'ja', //zh
            color: ' #f29543'
          },
          params: {
              curYear: dateObj.getFullYear(),
              curMonth: dateObj.getMonth(),
              curDate: dateObj.getDate(),
              curEventsDate: 'all'
          }
        }
      }
    },
    calendarParams () {
      let dateObj = new Date()
      if (inBrowser) {
          return window.VueCalendarBarEventBus.CALENDAR_EVENTS_DATA.params
      } else {
        return {
          curYear: dateObj.getFullYear(),
          curMonth: dateObj.getMonth(),
          curDate: dateObj.getDate(),
          curEventsDate: 'all'
        }
      }
    }
  },
  created () {
    if (this.calendarParams.curEventsDate !== 'all') {
      this.handleChangeCurDay(this.calendarParams.curEventsDate)
    }
  },
  methods: {
    handleChangeCurDay (date) {
      let events = this.events.filter(function(event) {
        return isEqualDateStr(event.date, date)
      })
      this.selectedDayEvents = {
        date: date,
        events: events
      }
      // if (events.length > 0) {
      //   this.selectedDayEvents = {
      //     date: date,
      //     events: events
      //   }
      // }
      this.$emit('day-changed', {
        date: date,
        events: events
      })
    },
    handleMonthChanged (yearMonth) {
      this.$emit('month-changed', yearMonth)
    }
  },
  watch: {
    calendarParams () {
      if (this.calendarParams.curEventsDate !== 'all') {
        let events = this.events.filter(event => {
          return isEqualDateStr(event.date, this.calendarParams.curEventsDate)
        })
        this.selectedDayEvents = {
          date: this.calendarParams.curEventsDate,
          events
        }
      } else {
        this.selectedDayEvents = {
          date: 'all',
          events: this.events
        }
      }
    },
    events () {
      this.selectedDayEvents = {
        date: 'all',
        events: this.events || []
      }
    }
  }
}
</script>
<style lang="less">
@base-orange: #f29543;
@white: #ffffff;
@gray: #e0e0e0;
@gray-dark: #b1b1b1;
@large-padding: 15px;
@small-padding: 10px;
@red_01: #ff2544;
@red_02: #fff5f3;
@main_text: #50656f;
@grey_03: #e7e7e7;

@icon-border-size: 1px;
@media screen and (min-width: 768px) {
  .__vev_calendar-wrapper{
    max-width: 1200px;
    margin: 0 auto;
    .cal-wrapper{
      width: 50%;
      padding: 100px 50px;
      .date-num{
        line-height: 50px;
      }
    }
    .events-wrapper{
      width: 50%;
      background-color: @base-orange;
      color: @white;
      padding: 40px 45px;
      position: absolute;
      left: 50%;
      top: 0;
      bottom: 0;
    }
  }
}
@media screen and (max-width: 768px) {
  .__vev_calendar-wrapper{
    .cal-wrapper{
      width: 100%;
      padding: 10px 5px;
      .date-num{
        line-height: 42px;
      }
    }
    .events-wrapper{
      width: 100%;
      margin-top: 10px;
      padding: 10px;
    }
  }
}
.__vev_calendar-wrapper{
  position: relative;
  overflow: hidden;
  width: 100%;
  *{
    box-sizing: border-box;
  }
  ::-webkit-scrollbar{
    width: 8px;
    height: 8px;
  }
  ::-webkit-scrollbar-track {
    box-shadow: inset 0 0 2px rgba(0,0,0,.2);
    border-radius: 5px;
  }
  ::-webkit-scrollbar-thumb {
    border-radius: 5px;
    background: rgba(0,0,0,.2);
  }
  .cal-wrapper{
    .cal-header{
      position: relative;
      width: 100%;
      background-color: @white;
      // box-shadow: 0 6px 5px rgba(0,0,0,.1);
      font-weight: 500;
      overflow: hidden;
      // padding-bottom: 10px;
      &>div{
        float: left;
        line-height: 20px;
        padding: @large-padding;
      }
      .title{
        width: 60%;
        text-align: center;
      }
      .l{
        text-align: left;
        width: 20%;
        cursor: pointer;
        user-select: none;
        -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
        & span {
          position: relative;
          bottom: -1px;
          display: block;
          padding-left: 30px;
          color: #8ea0a8;
          font-size: 13px;
        }
      }
      .r{
        text-align: right;
        width: 20%;
        cursor: pointer;
        user-select: none;
        -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
        & span {
          position: relative;
          bottom: -1px;
          display: block;
          padding-right: 30px;
          color: #8ea0a8;
          font-size: 13px;
        }
      }
    }
    .cal-body{
      width: 100%;
      .weeks{
        width: 100%;
        overflow: hidden;
        text-align: center;
        font-size: 1rem;
        .item{
          line-height: 50px;
          float: left;
          width: 14.285%;
        }
      }
      .dates{
        width: 100%;
        overflow: hidden;
        text-align: center;
        font-size: 1rem;
        .item{
          position: relative;
          float: left;
          display: block;
          width: 14.285%;
          cursor: default;
          -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
          .date-num{
            font-size: 1rem;
            position: relative;
            z-index: 3;
          }
          &.event{
            cursor: pointer;
          }
          &.selected-day{
            .is-event{
              background-color: @base-orange;
            }
          }
          .is-event{
            content: '';
            border: 1px solid @base-orange;
            background-color: #fff;
            border-radius: 50%;
            width: 36px;
            height: 36px;
            position: absolute;
            left: 50%;
            top: 50%;
            z-index: 1;
            margin-left: -18px;
            margin-top: -19px;
          }
          .is-today{
            content: '';
            background-color: @base-orange;
            border-radius: 50%;
            opacity: .8;
            width: 12px;
            height: 4px;
            position: absolute;
            left: 50%;
            top: 50%;
            z-index: 2;
            margin-left: -6px;
            margin-top: 8px;
          }
        }
      }
    }
  }
  .events-wrapper{
    border-radius: 10px;
    .cal-events{
      height: 95%;
      overflow-y: auto;
      padding: 0 5px;
      margin: 15px 0;
    }
    .date{
      max-width: 60%;
      min-width: 200px;
      text-align: center;
      color: @white;
      background-color: rgba(0, 0, 0, 0.2);
      border-radius: 20px;
      margin: 0 auto;
      font-size: 22px;
    }
    .event-item{
      padding: 5px 20px;
      margin-top: 15px;
      box-shadow: 0 3px 11px 2px rgba(0,0,0,.1);
      background-color: #fff;
      border-radius: 5px;
      color: #323232;
      position: relative;
      &:first-child{
        margin-top: 0;
      }
      .title{
        height: 40px;
        line-height: 40px;
        color: #323232;
        font-size: 16px;
        border-bottom: 1px solid #f2f2f2;
      }
      .time{
        position: absolute;
        right: 30px;
        top: 17px;
        color: #9b9b9b;
        font-size: 14px;
      }
      .desc{
        color: #9b9b9b;
        font-size: 14px;
        padding: 7px 0;
      }
    }
  }
  .arrow-left.icon {
    color: #000;
    position: absolute;
    left: 6%;
    margin-top: 10px;
  }
  .arrow-left.icon:before {
    content: '';
    position: absolute;
    left: 1px;
    top: -5px;
    width: 10px;
    height: 10px;
    border-top: solid @icon-border-size currentColor;
    border-right: solid @icon-border-size currentColor;
    -webkit-transform: rotate(-135deg);
            transform: rotate(-135deg);
  }
  .arrow-right.icon {
    color: #000;
    position: absolute;
    right: 6%;
    margin-top: 10px;
  }
  .arrow-right.icon:before {
    content: '';
    position: absolute;
    right: 1px;
    top: -5px;
    width: 10px;
    height: 10px;
    border-top: solid @icon-border-size currentColor;
    border-right: solid @icon-border-size currentColor;
    -webkit-transform: rotate(45deg);
            transform: rotate(45deg);
  }
  h3, p{
    margin: 0;
    padding: 0;
  }

  .cal-shows {
    background-color: @red_02;
    padding: 3px;
    margin: 0;
    border-bottom: 1px solid @grey_03;

    &--item {
      display: block;
      text-align: center;
      color: @main_text;
      font-size: 12px;
      line-height: 16px;
      padding: 2px 0;

      & b {
        font-weight: bold;
      }

      & strong {
        display: inline-block;
        margin-left: 16px;
        font-weight: normal;
        color: @red_01;
      }
    }
  }
}

</style>
