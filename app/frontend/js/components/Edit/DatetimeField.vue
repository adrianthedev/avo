<template>
  <edit-field-wrapper :field="field" :errors="errors" :index="index" :displayed-in="displayedIn">
    <flat-pickr
      ref="field-input"
      class="w-full"
      v-model="value"
      :enable-time="flatpickrConfig.enableTime"
      :config="flatpickrConfig"
      :placeholder="field.placeholder"
    />
    <template #extra>
      <a href="javascript:void(0);"
        class="p-4 inline-flex items-center"
        @click="setDateNow"
        v-text="nowLabel"
      />
      <span v-if="displayTimezone" class="px-4 items-center inline-flex text-gray-500">({{timezone}})</span>
    </template>
  </edit-field-wrapper>
</template>

<script>
// eslint-disable-next-line import/no-extraneous-dependencies
import '~/flatpickr/dist/flatpickr.css'
import { HasInputAppearance, IsFormField } from '@avo-hq/avo-js'
import flatPickr from 'vue-flatpickr-component'
import isNull from 'lodash/isNull'
import moment from 'moment-timezone'

export default {
  mixins: [IsFormField, HasInputAppearance],
  components: { flatPickr },
  data: () => ({
    value: '',
    timezone: '',
    appTimezone: 'UTC',
    enableTime: false,
    displayTimezone: false,
    flatpickrConfig: {
      enableTime: false,
      enableSeconds: false,
      // eslint-disable-next-line camelcase
      time_24hr: false,
      locale: {
        firstDayOfWeek: 0,
      },
      altInput: true,
      altInputClass: 'w-full',
    },
  }),
  computed: {
    submitFormat() {
      return 'YYYY-MM-DD'
    },
    nowLabel() {
      if (this.field.enable_time) { return 'Set now' }

      return 'Set today'
    },
  },
  methods: {
    setInitialConfig() {
      this.value = isNull(this.field.value) ? '' : moment(this.field.value)

      // enable timezone display
      if (this.field.enable_time && this.value) {
        this.displayTimezone = true
        this.flatpickrConfig.dateFormat = 'Y-m-d H:i:S'
        // eslint-disable-next-line camelcase
        this.flatpickrConfig.time_24hr = this.field.time_24hr
        this.timezone = Intl.DateTimeFormat().resolvedOptions().timeZone
        this.appTimezone = this.field.timezone

        if (this.value) {
          this.value = this.value.tz(this.timezone)
        }
      }

      if (this.value) {
        this.value = this.value.toDate()
      }

      // enable time settings
      this.flatpickrConfig.enableTime = this.field.enable_time
      this.flatpickrConfig.enableSeconds = this.field.enable_time

      this.flatpickrConfig.dateFormat = this.field.picker_format
      this.flatpickrConfig.altFormat = this.field.picker_format
      this.flatpickrConfig.altInputClass += ` ${this.inputClasses}`

      // set first day of the week
      this.flatpickrConfig.locale.firstDayOfWeek = this.field.first_day_of_week
    },
    getValue() {
      let value = moment(this.value)

      // Convert the time to the app's timezone
      if (this.field.enable_time) {
        value = value.tz(this.appTimezone)
      }

      if (this.field.enable_time) return value.toISOString()

      return value.format(this.submitFormat)
    },
    focus() {
      // No support for this at the moment.
    },
    setDateNow() {
      this.value = Date.now()
    },
  },
}
</script>
