<template>
  <div
    class="d-flex flex-column"
  >
    <b-card
      v-if="kind !== 'Text'"
      class="flex-grow-1 border-bottom border-light rounded-0"
    >
      <b-card-header
        header-tag="header"
        class="p-0 mb-3"
      >
        <h5
          class="mb-0"
        >
          {{ $t('general:general') }}
        </h5>
      </b-card-header>
      <b-card-body
        class="p-0"
      >
        <basic
          :item="item"
          @update-value="$emit('update-value', $event)"
        />
      </b-card-body>
    </b-card>

    <component
      :is="stepComponent"
      v-if="stepComponent"
      :item.sync="item"
      :edges.sync="edges"
      :out-edges="outEdges"
      :is-subworkflow="isSubworkflow"
      @update-value="$emit('update-value', $event)"
      @update-default-value="updateDefaultName"
    />
  </div>
</template>
<script>
import base from './base'
import basic from './basic'
import * as Configurators from './loader'

export default {
  components: {
    ...Configurators,
    basic,
  },

  extends: base,

  data () {
    return {
      collapse: {
        basic: true,
        configurator: true,
      },
    }
  },

  computed: {
    stepComponent () {
      return Configurators[this.kind]
    },

    kind () {
      const { kind, ref } = this.item.config

      if (kind === 'exec-workflow') {
        return 'ExecWorkflow'
      }

      if (kind === 'error-handler') {
        return 'ErrorHandler'
      }

      if (kind === 'visual' && ref === 'text') {
        return 'Text'
      }

      if (kind) {
        return kind.charAt(0).toUpperCase() + kind.slice(1)
      }

      return undefined
    },
  },

  methods: {
    updateDefaultName ({ value, force = false }) {
      if (force || this.item.config.defaultName || this.item.config.defaultName === undefined) {
        this.$emit('update-default-value', value)
      }
    },
  },
}
</script>
