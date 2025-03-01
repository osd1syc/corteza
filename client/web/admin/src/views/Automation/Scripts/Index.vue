<template>
  <b-container
    fluid="xl"
    class="d-flex flex-column h-100 pt-2 pb-3"
  >
    <c-content-header :title="$t('title')" />

    <b-card
      no-body
      class="flex-fill shadow-sm"
    >
      <template #header>
        <b-form
          @submit.prevent="search"
        >
          <b-row>
            <b-col
              cols="12"
              lg="6"
            >
              <b-form-group
                :label="$t('filter.searchQuery')"
                label-class="text-primary"
              >
                <b-form-input
                  v-model="filter.query"
                  size="sm"
                />
              </b-form-group>
            </b-col>
          </b-row>

          <b-row>
            <b-col
              cols="12"
              lg="6"
            >
              <b-form-group
                :label="$t('filter.incScriptsWithErrors', { count: totalScriptsWithErrors})"
                label-class="text-primary"
              >
                <c-input-checkbox
                  v-model="filter.incScriptsWithErrors"
                  size="sm"
                  :labels="checkboxLabel"
                  switch
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              lg="6"
            >
              <b-form-group
                :label="$t('filter.incScriptsWithTriggers', { count: totalScriptsWithTriggers})"
                label-class="text-primary"
              >
                <c-input-checkbox
                  v-model="filter.incScriptsWithTriggers"
                  size="sm"
                  :labels="checkboxLabel"
                  switch
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              lg="6"
            >
              <b-form-group
                :label="$t('filter.incScriptsWithIterator', { count: totalScriptsWithIterator})"
                label-class="text-primary"
              >
                <c-input-checkbox
                  v-model="filter.incScriptsWithIterator"
                  size="sm"
                  :labels="checkboxLabel"
                  switch
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              lg="6"
            >
              <b-form-group
                :label="$t('filter.incScriptsWithSecurity', { count: totalScriptsWithSecurity})"
                label-class="text-primary"
              >
                <c-input-checkbox
                  v-model="filter.incScriptsWithSecurity"
                  size="sm"
                  :labels="checkboxLabel"
                  switch
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              lg="6"
            >
              <b-form-group
                :label="$t('filter.absoluteTime')"
                label-class="text-primary"
              >
                <c-input-checkbox
                  v-model="filter.absoluteTime"
                  size="sm"
                  :labels="checkboxLabel"
                  switch
                />
              </b-form-group>
            </b-col>
          </b-row>
        </b-form>
      </template>

      <b-card-body
        class="p-0"
      >
        <b-table
          id="resource-list"
          hover
          responsive
          class="mb-0"
          head-variant="light"
          :items="filtered"
          :fields="fields"
          :empty-text="$t('admin:general.notFound')"
          show-empty
          no-sort-reset
        >
          <template #table-busy>
            <div class="text-center m-5">
              <div>
                <b-spinner
                  small
                  class="align-middle m-2"
                />
              </div>
              <div>{{ $t('loading') }}</div>
            </div>
          </template>
          <template #cell(label)="{}" />
          <template #cell(name)="{ item: { label, name, errors, description, ...r }, toggleDetails }">
            <div>
              <span
                v-if="label"
              >
                {{ label }}
              </span>

              <span
                v-else
                class="text-secondary"
              >{{ $t('labelMissing') }}
              </span>

              <b-badge
                v-if="r.security"
                class="rounded m-1 py-1 px-2 pointer"
                variant="primary"
                @click="toggleDetails"
              >
                {{ $t('flags.security') }}
              </b-badge>

              <b-badge
                v-if="r.triggers"
                class="rounded m-1 py-1 px-2 pointer"
                variant="primary"
                @click="toggleDetails"
              >
                {{ $t('flags.triggers') }}
              </b-badge>

              <b-badge
                v-if="r.iterator"
                class="rounded m-1 py-1 px-2 pointer"
                variant="primary"
                @click="toggleDetails"
              >
                {{ $t('flags.iterator') }}
              </b-badge>
            </div>

            <p
              v-if="description"
              class="text-secondary"
            >
              {{ description }}
            </p>

            <div><small><code>{{ name }}</code></small></div>

            <b-alert
              v-for="(error, i) in errors"
              :key="i"
              show
              variant="warning"
            >
              {{ error }}
            </b-alert>
          </template>

          <template #row-details="{ item: r }">
            <b-card>
              <pre>{{ r.triggers }}</pre>
              <pre>{{ r.iterator }}</pre>
              <pre>{{ r.security }}</pre>
            </b-card>
          </template>

          <template #cell(updatedAt)="{ value }">
            <time
              v-b-tooltip.noninteractive.hover="{ title: value, container: '#body' }"
              :datetime="value.toISOString()"
            >
              {{ filter.absoluteTime ? value : value.fromNow() }}
            </time>
          </template>
        </b-table>
      </b-card-body>
    </b-card>
  </b-container>
</template>

<script>
import listHelpers from 'corteza-webapp-admin/src/mixins/listHelpers'
import moment from 'moment'

export default {
  mixins: [
    listHelpers,
  ],

  i18nOptions: {
    namespaces: 'automation.scripts',
    keyPrefix: 'list',
  },

  data () {
    return {
      id: 'automation',

      items: [],

      filter: {
        query: '',
        incScriptsWithErrors: false,
        incScriptsWithTriggers: false,
        incScriptsWithIterator: false,
        incScriptsWithSecurity: false,
        absoluteTime: false,
      },

      fields: [
        {
          key: 'name',
          label: 'Name',
          sortable: true,
        },
        {
          key: 'updatedAt',
          sortable: true,
          tdClass: 'text-right text-nowrap',
          formatter: (v) => moment(v),
        },
      ].map(c => ({
        // Generate column label translation key
        label: this.$t(`columns.${c.key}`),
        ...c,
      })),

      checkboxLabel: {
        on: this.$t('general:label.general.yes'),
        off: this.$t('general:label.general.no'),
      },
    }
  },

  computed: {
    filtered () {
      const {
        query,
        incScriptsWithErrors,
        incScriptsWithTriggers,
        incScriptsWithIterator,
        incScriptsWithSecurity,
      } = this.filter
      const lcQuery = query.toLocaleLowerCase()

      return this.items
        .filter(({ name, label }) => (lcQuery.length === 0 || (name + ' ' + label).toLocaleLowerCase().indexOf(lcQuery) > -1))
        .filter(({ errors }) => (incScriptsWithErrors === false || (errors && errors.length > 0)))
        .filter(({ triggers }) => (incScriptsWithTriggers === false || !!triggers))
        .filter(({ iterator }) => (incScriptsWithIterator === false || !!iterator))
        .filter(({ security }) => (incScriptsWithSecurity === false || !!security))
    },

    totalScriptsWithErrors () {
      return this.items.filter(({ errors }) => (errors && errors.length > 0)).length
    },

    totalScriptsWithSecurity () {
      return this.items.filter(({ security }) => (security)).length
    },

    totalScriptsWithTriggers () {
      return this.items.filter(({ triggers }) => (triggers)).length
    },

    totalScriptsWithIterator () {
      return this.items.filter(({ iterator }) => (iterator)).length
    },
  },

  created () {
    this.procListResults(this.$SystemAPI.automationList(this.encodeListParams()))
      .then(set => { this.items = set || [] })
  },

  methods: {
    events (tt) {
      const ee = []

      if (!Array.isArray(tt) || tt.length === 0) {
        return ee
      }

      tt.forEach(({ events }) => ee.push(...(events || [])))
      return ee.filter((v, i) => ee.indexOf(v) === i)
    },
  },
}
</script>
<style lang="scss">
.pointer {
  cursor: pointer;
}
</style>
