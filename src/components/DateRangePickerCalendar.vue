<template>
  <div class="daterangepicker-calendar">
    <div class="d-flex align-items-center">
      <div class="p-1" :class="calendarIndex == 1 ? '' : 'invisible'">
        <button type="button" class="btn btn-sm btn-light" @mousedown.prevent @click="goToPrevMonth">
          <FontAwesomeIcon :icon="['fa', 'caret-left']" fixed-width />
        </button>
      </div>
      <div class="p-1 col text-center">
        {{ displayMonth.format('MMMM YYYY') }}
      </div>
      <div class="p-1" :class="calendarIndex == calendarCount ? '' : 'invisible'">
        <button type="button" class="btn btn-sm btn-light" @mousedown.prevent @click="goToNextMonth">
          <FontAwesomeIcon :icon="['fa', 'caret-right']" fixed-width />
        </button>
      </div>
    </div>
    <div class="d-flex justify-content-between text-center daterangepicker-calendar-row">
      <div v-for="day in daysOfFirstWeek" :key="day.format('D')" class="col-day">{{ day.format('ddd') }}</div>
    </div>

    <div class="d-flex flex-wrap justify-content-between text-center daterangepicker-calendar-row">
      <div v-for="day in days" :key="day.format('M-D')" class="col-day"
           :style="{backgroundColor: calculateColor(day)}"
           :class="dayClass(day)" @mouseover="dayMouseOver(day)" @mousedown.prevent @click="dayClick(day)">{{ day.format('D') }}</div>
    </div>
  </div>
</template>

<script>
  import {faCaretLeft} from '@fortawesome/free-solid-svg-icons/faCaretLeft'
  import {faCaretRight} from '@fortawesome/free-solid-svg-icons/faCaretRight'
  import { library } from '@fortawesome/fontawesome-svg-core'
  import _ from 'lodash'
  import Color from 'chartjs-color'
  library.add(faCaretLeft, faCaretRight)
    import moment from 'moment'
    import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
    export default {

        components: {
        FontAwesomeIcon
      },
        props: ['calendarIndex', 'calendarCount', 'month', 'startDate', 'endDate', 'compare', 'startDateCompare', 'endDateCompare', 'step',
          'heatMapData',
          'heatMapDataXName',
          'heatMapDataYName',
        ],
        data: () => {
            return {}
        },
        computed: {
            displayMonth: function() {
                return moment.utc(this.month).add(this.calendarIndex - (this.calendarCount - 0), 'month')
            },
            days: function() {
                let startDay = moment.utc(this.displayMonth).startOf('isoWeek')
                let endDay = moment.utc(this.displayMonth).endOf('month').endOf('isoWeek').startOf('day').add(1, 'day')
                let nDays = moment.duration(endDay.diff(startDay)).asDays()

                let days = []
                let day = 0
                while (day < nDays) {
                    days.push(moment.utc(startDay).add(day, 'day'))
                    day++
                }
                return days
            },
            daysOfFirstWeek: function() {
                return this.days.slice(0, 7)
            },
            heatMapMins(){
              let heatMapMins = {}
              _.each(this.heatMapData, (e) => {
                let month = moment(e[this.heatMapDataXName]).month()
                let year = moment(e[this.heatMapDataXName]).year()
                heatMapMins[year] = heatMapMins[year] || {}
                heatMapMins[year][month] = heatMapMins[year][month] || Infinity
                heatMapMins[year][month] = Math.min(e[this.heatMapDataYName], heatMapMins[year][month])
              })
              return heatMapMins
            },
          heatMapMaxs(){
            let heatMapMaxs = {}
            _.each(this.heatMapData, (e) => {
              let month = moment(e[this.heatMapDataXName]).month()
              let year = moment(e[this.heatMapDataXName]).year()
              heatMapMaxs[year] = heatMapMaxs[year] || {}
              heatMapMaxs[year][month] = heatMapMaxs[year][month] || Infinity
              heatMapMaxs[year][month] = Math.max(e[this.heatMapDataYName], heatMapMaxs[year][month])
            })
            return heatMapMaxs
          },
        },
        methods: {
          calculateColor(day){
            if(!this.heatMapData || this.heatMapData.length <= 0){
              return 'transparent'
            }
            // console.log(day, this.heatMapMin, this.heatMapMax)
            // console.log(`looking for `, day.format('YYYY-MM-DD'), this.heatMapData, `field name:`, this.heatMapDataXName)
            let heatValue = _.find(this.heatMapData, {[this.heatMapDataXName]: day.format('YYYY-MM-DD')})
            if(!heatValue) return 'transparent'
            heatValue = heatValue[this.heatMapDataYName]
            let heatMapMin = this.heatMapMins[day.year()][day.month()]
            let heatMapMax = this.heatMapMaxs[day.year()][day.month()]
            let d = (heatMapMax - heatMapMin)

            let ratio = (heatValue - heatMapMin) / d
            return Color().blue(0).green(255 * ratio).red(255 * (1 - ratio)).lighten(.6).hslString()
          },
            dayClass: function(day) {
                let classes = []

                // Hide days overflowing
                if (!day.isBetween(this.displayMonth, moment.utc(this.displayMonth).endOf('month'), 'days', '[]')) {
                    classes.push('invisible')
                }
                // Class for days between startDate & endDate or is startDate (in case of startDate after endDate)
                if (day.isBetween(this.startDate, this.endDate, 'days', '[]') || day.isSame(this.startDate)) {
                    classes.push('daterangepicker-range')
                    this.startDate.startOf('day')
                    this.endDate.startOf('day')
                    //console.log(day, this.startDate.startOf('day'), this.endDate.startOf('day'));
                    if(day.isSame(this.startDate)){
                        classes.push('daterangepicker-range-first')
                    }
                    if(day.isSame(this.endDate)){
                        classes.push('daterangepicker-range-last')
                    }
                }
                // Class for days between startDateCompare & endDateCompare or is startDateCompare (in case of startDateCompare after endDateCompare)
                if (this.compare && (day.isBetween(this.startDateCompare, this.endDateCompare, 'days', '[]') || day.isSame(this.startDateCompare))) {
                    classes.push('daterangepicker-range-compare')
                }
                // Class for cursor if the step is selecting something
                if (this.step != null) {
                    classes.push('daterangepicker-cursor-pointer')
                }

                return classes.join(' ')
            },
            dayMouseOver: function(day) {
                if (this.step != null) {
                    this.selectDate(day)
                }
            },
            dayClick: function(day) {

                if (this.step != null) {
                    this.nextStep()
                } else {
                    this.$emit('dayClickNoStep');
                    this.selectDate(day)
                    this.nextStep()
                    this.selectDate(day)
                }
            },
            goToPrevMonth: function() {
                this.$emit('goToPrevMonth')
            },
            goToNextMonth: function() {
                this.$emit('goToNextMonth')
            },
            selectDate: function(date) {
                this.$emit('selectDate', date)
            },
            nextStep: function() {
                this.$emit('nextStep')
            }
        },
        watch: {},
        filters: {},
    }
</script>

<style lang="scss">
  .daterangepicker-calendar-row {
    font-size: 14px;
    .daterangepicker-range {
      background-color: #17a2b8 !important;
      color: #ffffff;
      &.daterangepicker-range-first {
        border-bottom-left-radius: 25px;
        border-top-left-radius: 25px;
      }
      &.daterangepicker-range-last {
        border-bottom-right-radius: 25px;
        border-top-right-radius: 25px;
      }
    }
  }

  .col-day {
    width: 14.28571428571429%;
    padding: 0.5rem 0;
    white-space: nowrap;
    cursor: pointer;
    margin-bottom: 2px;
  }



  .daterangepicker-range-compare {
    background-color: #ff9307;
    color: #ffffff;
  }

  .daterangepicker-range.daterangepicker-range-compare {
    background: linear-gradient(0, #ff9307 50%, #17a2b8 50%);
  }

  .daterangepicker-cursor-pointer {
    cursor: pointer;
  }
</style>
