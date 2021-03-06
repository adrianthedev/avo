<template>
  <filter-wrapper :name="filter.name" :index="index">
      <div class="flex items-center">
        <div class="space-y-2">
          <template v-for="(name, value, index) in filter.options">
            <div
              :key="index"
              class="flex items-center"
              @click="toggleOption(value)"
            >
              <input
                type="checkbox"
                class="mr-2 text-lg h-4 w-4"
                :id="name"
                :name="name"
                :checked="optionToggled(value)"
              />
              <label class="font text-gray-700 text-sm">{{ name }}</label>
            </div>
          </template>
        </div>
      </div>
  </filter-wrapper>
</template>

<script>
import { HasInputAppearance } from '@avo-hq/avo-js'

export default {
  mixins: [HasInputAppearance],
  data: () => ({
    selected: [],
  }),
  props: [
    'filter',
    'appliedFilters',
    'index',
  ],
  computed: {
    filterClass() {
      if (this.filter && this.filter.filter_class) {
        return this.filter.filter_class
      }

      return ''
    },
    filterValue() {
      const value = {}

      Object.keys(this.filter.options).forEach((option) => {
        value[option] = this.optionToggled(option)
      })

      return value
    },
  },
  methods: {
    toggleOption(option) {
      if (this.optionToggled(option)) {
        this.selected = this.selected.filter((currentOption) => currentOption !== option)
      } else {
        this.selected.push(option)
      }

      this.changeFilter()
    },
    optionToggled(option) {
      return this.selected.indexOf(option) > -1
    },
    changeFilter() {
      return this.$emit('change-filter', { [this.filterClass]: this.filterValue })
    },
    applySelection(selection) {
      Object.keys(selection).forEach((option) => {
        if (selection[option]) {
          this.selected.push(option)
        }
      })
    },
    setInitialValue() {
      const presentFilterValue = this.appliedFilters[this.filterClass]

      if (presentFilterValue) {
        this.applySelection(presentFilterValue)
      } else if (this.filter.default) {
        this.applySelection(this.filter.default)
      }
    },
  },
  mounted() {
    this.setInitialValue()
  },
}
</script>
