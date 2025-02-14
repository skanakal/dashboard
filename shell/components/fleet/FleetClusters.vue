<script>
import ResourceTable from '@shell/components/ResourceTable';
import { STATE, NAME, AGE, FLEET_SUMMARY } from '@shell/config/table-headers';
import { FLEET, MANAGEMENT } from '@shell/config/types';

export default {
  components: { ResourceTable },

  props: {
    rows: {
      type:     Array,
      required: true,
    },

    schema: {
      type:    Object,
      default: null,
    },

    loading: {
      type:    Boolean,
      default: false,
    },
    useQueryParamsForSimpleFiltering: {
      type:    Boolean,
      default: false
    }
  },

  computed: {
    MANAGEMENT_CLUSTER() {
      return MANAGEMENT.CLUSTER;
    },

    headers() {
      const out = [
        STATE,
        NAME,
        {
          name:     'bundlesReady',
          labelKey: 'tableHeaders.bundlesReady',
          value:    'status.display.readyBundles',
          sort:     'status.summary.ready',
          search:   false,
        },
        {
          name:     'reposReady',
          labelKey: 'tableHeaders.reposReady',
          value:    'status.readyGitRepos',
          sort:     'status.summary.ready',
          search:   false,
        },
        FLEET_SUMMARY,
        {
          name:          'lastSeen',
          labelKey:      'tableHeaders.lastSeen',
          value:         'status.agent.lastSeen',
          sort:          'status.agent.lastSeen',
          search:        false,
          formatter:     'LiveDate',
          formatterOpts: { addSuffix: true },
          width:         120,
        },
        AGE,
      ];

      return out;
    },

    pagingParams() {
      const schema = this.$store.getters[`management/schemaFor`](FLEET.CLUSTER);

      return {
        singularLabel: this.$store.getters['type-map/labelFor'](schema),
        pluralLabel:   this.$store.getters['type-map/labelFor'](schema, 99),
      };
    },
  }
};
</script>

<template>
  <ResourceTable
    v-bind="$attrs"
    :schema="schema"
    :headers="headers"
    :rows="rows"
    :loading="loading"
    :use-query-params-for-simple-filtering="useQueryParamsForSimpleFiltering"
    key-field="_key"
  >
    <template #cell:workspace="{row}">
      <span v-if="row.type !== MANAGEMENT_CLUSTER && row.metadata.namespace">{{ row.metadata.namespace }}</span>
      <span
        v-else
        class="text-muted"
      >&mdash;</span>
    </template>

    <template #cell:reposReady="{row}">
      <span
        v-if="!row.repoInfo"
        class="text-muted"
      >&mdash;</span>
      <span
        v-else-if="row.repoInfo.unready"
        class="text-warning"
      >{{ row.repoInfo.ready }}/{{ row.repoInfo.total }}</span>
      <span v-else>{{ row.repoInfo.total }}</span>
    </template>

    <template #cell:bundlesReady="{row}">
      <span
        v-if="row.bundleInfo.noValidData"
        class="text-muted"
      >&mdash;</span>
      <span
        v-else-if="row.bundleInfo.ready !== row.bundleInfo.total"
        class="text-warning"
      >{{ row.bundleInfo.ready }}/{{ row.bundleInfo.total }}</span>
      <span
        v-else
        :class="{'text-error': !row.bundleInfo.total}"
      >{{ row.bundleInfo.total }}</span>
    </template>
  </ResourceTable>
</template>
