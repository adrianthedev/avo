<template>
  <div class="space-x-2">
    <router-link
      :to="{
        name: 'show',
        params: {
          resourceName: resourcePath,
          resourceId: resource.id,
        },
        query:{
          viaResourceName: viaResourceName,
          viaResourceId: viaResourceId,
        }
      }"
      v-tooltip="`View ${this.resourceNameSingular}`"
      data-control="view"
    >
      <eye-icon :class="iconClasses"/>
    </router-link>
    <router-link
      :to="{
        name: 'edit',
        params: {
          resourceName: resourcePath,
          resourceId: resource.id,
        },
        query: {
          viaResourceName: viaResourceName,
          viaResourceId: viaResourceId,
        },
      }"
      v-tooltip="`Edit ${this.resourceNameSingular}`"
      data-control="edit"
    >
      <edit-icon :class="iconClasses"/>
    </router-link>
    <a href="javascript:void(0);"
      @click="openDetachModal"
      v-tooltip="`Detach ${this.resourceNameSingular}`"
      data-control="detach"
      v-if="relationship === 'has_and_belongs_to_many'"
    >
      <trash-icon :class="iconClasses"/>
    </a>
    <a href="javascript:void(0);"
      @click="openDeleteModal"
      v-tooltip="`Delete ${this.resourceNameSingular}`"
      data-control="delete"
      v-else
    >
      <trash-icon :class="iconClasses"/>
    </a>
  </div>
</template>

<script>
import { Api } from '@/js/Avo'
import DealsWithHasManyRelations from '@/js/mixins/deals-with-has-many-relations'
import DealsWithResourceLabels from '@/js/mixins/deals-with-resource-labels'
import Modal from '@/js/components/Modal.vue'
import isUndefined from 'lodash/isUndefined'

export default {
  mixins: [DealsWithHasManyRelations, DealsWithResourceLabels],
  data: () => ({
    iconClasses: 'text-gray-500 h-6 hover:text-gray-600',
  }),
  props: [
    'resource',
    'resourceName',
    'viaResourceName',
    'viaResourceId',
    'field',
  ],
  computed: {
    afterSuccessPath() {
      if (!isUndefined(this.viaResourceName)) return `/resources/${this.viaResourceName}/${this.viaResourceId}`

      return `/resources/${this.resourceName}`
    },
  },
  methods: {
    async deleteResource() {
      await Api.delete(`/avocado/avocado-api/${this.resourcePath}/${this.resource.id}`)

      this.$modal.hideAll()

      this.$emit('resource-deleted')
    },
    async detachResource() {
      await Api.post(`/avocado/avocado-api/${this.viaResourceName}/${this.viaResourceId}/detach/${this.resourcePath}/${this.resource.id}`)

      this.$modal.hideAll()

      this.$emit('resource-deleted')
    },
    openDeleteModal() {
      this.$modal.show(Modal, {
        heading: `Delete ${this.resourceNameSingular}`,
        text: 'Are you sure?',
        confirmAction: this.deleteResource,
      })
    },
    openDetachModal() {
      this.$modal.show(Modal, {
        text: `Are you sure you want to detach this ${this.resourceNameSingular.toLowerCase()}?`,
        confirmAction: this.detachResource,
      })
    },
  },
  mounted() {},
}
</script>