<template>
  <div class="d-flex daterangepicker-row">
    <!-- Calendars -->
    <div class="daterangepicker-col" v-for="calendarIndex in calendarCount" :key="calendarIndex">
      <date-range-picker-calendar
              :calendarIndex="calendarIndex"
              :calendarCount="calendarCount"
              :month="month"
              :startDate="startDate"
              :endDate="endDate"
              :compare="compare"
              :startDateCompare="startDateCompare"
              :endDateCompare="endDateCompare"
              :step="step"
              :heatMapData="heatMapData"
              :heatMapDataXName="heatMapDataXName"
              :heatMapDataYName="heatMapDataYName"
              v-on:goToPrevMonth="goToPrevMonth"
              v-on:goToNextMonth="goToNextMonth"
              v-on:selectDate="selectDate"
              v-on:nextStep="nextStep"
			  v-on:dayClickNoStep="onDayClickNoStep"
      />
    </div>

    <!-- Right form -->
    <div class="daterangepicker-col">
      <div class="form-group">
        <select class="custom-select" :class="compare ? 'daterangepicker-range-border' : ''" v-model="rangeSelect">
          <option v-for="(range, rangeKey) in ranges" :key="rangeKey" :value="rangeKey">{{ range.label }}</option>
          <option value="custom">Custom range</option>
        </select>
      </div>
      <div class="form-group form-inline flex-nowrap">
        <input type="text" class="form-control w-100 daterangepicker-date-input"
               ref="startDate"
               :value="startDate | dateFormat(dateFormatPattern)"

        >
        <span class="mx-2">
        <FontAwesomeIcon :icon="['fa', 'caret-right']" fixed-width />
        </span>
        <input type="text" class="form-control w-100 daterangepicker-date-input"
               ref="endDate"
               :value="endDate | dateFormat(dateFormatPattern)"

        >
      </div>
      <div class="form-group" v-if="allowCompare">
        <div class="custom-control custom-checkbox">
          <input type="checkbox" class="custom-control-input" :id="'date-range-picker-compare-' + _uid" v-model="compare">
          <label class="custom-control-label" :for="'date-range-picker-compare-' + _uid">Compare</label>
        </div>
      </div>
      <div v-if="compare">
        <div class="form-group">
          <select class="custom-select" :class="compare ? 'daterangepicker-range-border compare' : ''" v-model="rangeSelectCompare">
            <option v-for="(range, rangeKey) in ranges" :key="rangeKey" :value="rangeKey">{{ range.label }}</option>
            <option value="custom">Custom range</option>
          </select>
        </div>
        <div class="form-group form-inline flex-nowrap">
          <input type="text" class="form-control w-100 daterangepicker-date-input compare"
                 ref="startDateCompare"
                 :value="startDateCompare | dateFormat(dateFormatPattern)"
                 @keyup.enter="inputDate"
          >
          <span class="mx-2">
          <FontAwesomeIcon :icon="['fa', 'caret-right']" fixed-width />
          </span>
          <input type="text" class="form-control w-100 daterangepicker-date-input compare"
                 ref="endDateCompare"
                 :value="endDateCompare | dateFormat(dateFormatPattern)"
                 @keyup.enter="inputDate"
          >
        </div>
      </div>
      <div class="form-group form-inline justify-content-end mb-0 buttons">
        <button type="button" class="btn btn-light" @click="cancel">Cancel</button>
        <button type="button" class="btn btn-primary ml-2" :disabled="step != null" @click="submit">Submit</button>
      </div>
    </div>
  </div>
</template>

<script>
    import DateRangePickerCalendar from './DateRangePickerCalendar'
    import {faCaretLeft} from '@fortawesome/free-solid-svg-icons/faCaretLeft'
    import {faCaretRight} from '@fortawesome/free-solid-svg-icons/faCaretRight'
    import { library } from '@fortawesome/fontawesome-svg-core'
    library.add(faCaretLeft, faCaretRight)
    import moment from 'moment'

    import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'

    export default {
      components: {
        DateRangePickerCalendar,
        FontAwesomeIcon
      },
        props: {
            calendarCount: {
                type: Number,
                default: 2
            },
            allowCompare: {
                type: Boolean,
                default: true
            },
            ranges: {
                type: Object,
                default: () => {
                    return {
                        today: {
                            label: 'Today',
                            startDate: moment.utc().startOf('day'),
                            endDate: moment.utc().endOf('day')
                        },
                        lastWeek: {
                            label: 'Last Week',
                            startDate: moment.utc().subtract(1, 'week').startOf('isoWeek'),
                            endDate: moment.utc().subtract(1, 'week').endOf('isoWeek').startOf('day')
                        },
                        currentMonth: {
                            label: 'Current month',
                            startDate: moment.utc().startOf('month'),
                            endDate: moment.utc().endOf('month').startOf('day')
                        },
                        lastMonth: {
                            label: 'Last month',
                            startDate: moment.utc().subtract(1, 'month').startOf('month'),
                            endDate: moment.utc().subtract(1, 'month').endOf('month').startOf('day')
                        },
                        lastYear: {
                          label: 'Last year',
                          startDate: moment.utc().subtract(1, 'year').startOf('year'),
                          endDate: moment.utc().subtract(1, 'year').endOf('year').startOf('day')
                        },
                    }
                }
            },
            defaultRangeSelect: {
                type: String,
                default: 'currentMonth'
            },
            defaultRangeSelectCompare: {
                type: String,
                default: 'lastMonth'
            },
            dateFormatPattern: {
                type: String,
                default: 'DD-MM-YYYY'
            },
            timeFormatPattern: {
                type: String,
                default: 'hh:mm'
            },
            heatMapData: {
              type: Array,
              default: () => [
                {
                  x: moment().startOf('day').format('YYYY-MM-DD'),
                  y: 120
                },
                {
                  x: moment().subtract(1, 'day').startOf('day').format('YYYY-MM-DD'),
                  y: 150
                },
                {
                  x: moment().subtract(3, 'day').startOf('day').format('YYYY-MM-DD'),
                  y: 110
                },
                {
                  x: moment().subtract(5, 'day').startOf('day').format('YYYY-MM-DD'),
                  y: 130
                }
              ]
            },
            heatMapDataXName: {
              type: String,
              default: 'x'
            },
            heatMapDataYName: {
              type: String,
              default: 'y'
            },
        },
        data() {
            return {
                startDate: moment.utc(),
                endDate: moment.utc(),
                startDateCompare: moment.utc(),
                endDateCompare: moment.utc(),
                rangeSelect: null,
                rangeSelectCompare: null,
                compare: false,
                month: moment.utc().startOf('month'),
                step: null
            }
        },
        computed: {
            nextMonth() {
                return moment.utc(this.month).add(1, 'month')
            },
            // For multi prop watchers
            range() {
                return this.startDate, this.endDate
            },
            rangeCompare() {
                return this.startDateCompare, this.endDateCompare
            }
        },
        methods: {
            onDayClickNoStep(){
                this.step = 'selectStartDate';
			},
            goToPrevMonth() {
                this.month = moment.utc(this.month).subtract(1, 'month')
            },
            goToNextMonth() {
                this.month = moment.utc(this.month).add(1, 'month')
            },
            selectRange(rangeKey) {
                let predefinedRange = false

                // Predefined ranges
                for (const _rangeKey of Object.keys(this.ranges)) {
                    const range = this.ranges[_rangeKey]
                    if (rangeKey == _rangeKey) {
                        predefinedRange = true

                        if (!this.startDate.isSame(range.startDate)) {
                            this.startDate = moment.utc(range.startDate)
                        }
                        if (!this.endDate.isSame(range.endDate)) {
                            this.endDate = moment.utc(range.endDate)
                        }
                    }
                }

                // Custom range
                if (!predefinedRange && this.step == null) {
                    this.step = 'selectStartDate'
                    //this.$refs.startDate.focus()
                }
            },
            selectRangeCompare(rangeKey) {
                let predefinedRange = false

                // Predefined ranges
                for (const _rangeKey of Object.keys(this.ranges)) {
                    const range = this.ranges[_rangeKey]
                    if (rangeKey == _rangeKey) {
                        predefinedRange = true

                        if (!this.startDateCompare.isSame(range.startDate)) {
                            this.startDateCompare = moment.utc(range.startDate)
                        }
                        if (!this.endDateCompare.isSame(range.endDate)) {
                            this.endDateCompare = moment.utc(range.endDate)
                        }
                    }
                }

                // Custom range
                if (!predefinedRange && this.step == null) {
                    this.step = 'selectStartDateCompare'
                    //this.$refs.startDateCompare.focus()
                }
            },
            selectDate(date){
                if (this.step == 'selectStartDate') {
                    this.startDate = date
                } else if (this.step == 'selectEndDate') {
                    this.endDate = date
                } else if (this.step == 'selectStartDateCompare') {
                    this.startDateCompare = date
                } else if (this.step == 'selectEndDateCompare') {
                    this.endDateCompare = date
                }
            },
            // Step flow for date range selections
            nextStep() {
                if (this.step == 'selectStartDate') {
                    this.step = 'selectEndDate'
                    //this.$refs.endDate.focus()
                } else if (this.step == 'selectEndDate') {
                    this.step = null
                    //this.$refs.endDate.blur()
                } else if (this.step == 'selectStartDateCompare') {
                    this.step = 'selectEndDateCompare'
                    //this.$refs.endDateCompare.focus()
                } else if (this.step == 'selectEndDateCompare') {
                    this.step = null
                    //this.$refs.endDateCompare.blur()
                }
            },
            // Try to update the step date from an input value
            inputDate(input) {
                let date = moment.utc(input.target.value, 'YYYY-MM-DD')
                if (date.isValid()) {
                    this.selectDate(date)
                }
                this.nextStep()
            },
            // Submit button
            submit() {
                this.$emit('submit', {
                    startDate: this.startDate,
                    endDate: this.endDate,
                    compare: this.compare,
                    startDateCompare: this.startDateCompare,
                    endDateCompare: this.endDateCompare
                })
            },
            // Cancel button
            cancel() {
                this.$emit('cancel')
            }
        },
        watch: {
            rangeSelect(rangeKey) {
                this.selectRange(rangeKey)
            },
            rangeSelectCompare(rangeKey) {
                this.selectRangeCompare(rangeKey)
            },
            range() {
                let predefinedRange = false

                // Predefined ranges
                for (const rangeKey of Object.keys(this.ranges)) {
                    const range = this.ranges[rangeKey]
                    if (this.startDate.isSame(range.startDate) && this.endDate.isSame(range.endDate)) {
                        predefinedRange = true
                        if (this.rangeSelect != rangeKey) {
                            this.rangeSelect = rangeKey
                        }
                    }
                }

                // Custom range
                if (!predefinedRange) {
                    if (this.rangeSelect != 'custom') {
                        this.rangeSelect = 'custom'
                    }
                }
            },
            rangeCompare() {
                let predefinedRange = false

                // Predefined ranges
                for (const rangeKey of Object.keys(this.ranges)) {
                    const range = this.ranges[rangeKey]
                    if (this.startDateCompare.isSame(range.startDate) && this.endDateCompare.isSame(range.endDate)) {
                        predefinedRange = true
                        if (this.rangeSelectCompare != rangeKey) {
                            this.rangeSelectCompare = rangeKey
                        }
                    }
                }

                // Custom range
                if (!predefinedRange) {
                    if (this.rangeSelectCompare != 'custom') {
                        this.rangeSelectCompare = 'custom'
                    }
                }
            },
			startDate(){
                this.month = moment.utc(this.startDate).startOf('month')
			}
        },
        filters: {
            dateFormat(value, format) {
                return value ? value.format(format) : ''
            }
        },
        created() {
            // Initialize ranges
            this.rangeSelect = this.defaultRangeSelect
            this.rangeSelectCompare = this.defaultRangeSelectCompare
        }
    }
</script>

<style lang="scss">
  .daterangepicker-row {
    margin: -0.5rem;
  }

  .daterangepicker-col {
    padding: 0.5rem;
    flex-basis: 100%;
    position: relative;
    .buttons {
      position: absolute;
      right: 10px;
      bottom: 10px;
    }
  }

  /* Make sure that the full date (YYYY-MM-DD) is displayed */
  .daterangepicker-date-input {
    min-width: 120px;
  }

  /* Select menus border */
  .daterangepicker-range-border {
    border-color: #17a2b8 !important;
  }

  .daterangepicker-range-border.compare {
    border-color: #ff9307 !important;
  }

  /* Date input focus */
  .daterangepicker-date-input:focus {
    border-color: #17a2b8 !important;
    box-shadow: 0 0 0 0.2rem rgba(23, 162, 184, 0.25) !important;
  }

  .daterangepicker-date-input.compare:focus {
    border-color: #ff9307 !important;
    box-shadow: 0 0 0 0.2rem rgba(255, 147, 7, 0.25) !important;
  }
</style>
