<template>
  <div class="vbr-widget">
    <ErrorNotice v-if="error" :error="error"></ErrorNotice>

    <ResponsiveTable v-else>
      <DataTable
        class="vbr-widget-standings"
        :columns="columns"
        :rows="convertedData"
        :sort="sort"
        :is-loading="isLoading"
        @sort="onSort"
      >
        <template v-slot:cell-name="{ row }">
          <ImageBase
            class="vbr-widget-image"
            :src="`https://jegkorongszovetseg.hu/_upload/editor/db/team-logos/211/${row.id}.png`"
          />
          {{ row.name }}
        </template>
      </DataTable>
    </ResponsiveTable>
  </div>
</template>

<script>
import convert from '../../services/convert';
import { SortService } from '../../services/sort-service';
import DataTable from '../DataTable';
import ErrorNotice from '../ErrorNotice';
import ResponsiveTable from '../ResponsiveTable';
import ImageBase from '../ImageBase';
import { fetchVBRData } from '../../services/http-sevices';

export default {
  name: 'Standings',

  components: {
    ImageBase,
    DataTable,
    ErrorNotice,
    ResponsiveTable
  },

  props: {
    championshipId: {
      type: String,
      require: true
    },

    division: {
      type: String,
      require: true
    },

    lang: {
      type: String,
      default: 'hu'
    }
  },

  data() {
    return {
      error: '',
      columns: {
        index: {
          label: '#',
          class: 'text-left'
        },
        name: {
          label: 'table.team.short',
          tooltip: 'table.team.tooltip',
          sortable: true,
          class: 'text-left w-auto'
        },
        m: {
          label: 'M',
          tooltip: 'table.team.tooltip',
          sortable: true
        },
        p3: {
          label: 'GY',
          tooltip: 'table.team.tooltip',
          sortable: true
        },
        p2: {
          label: 'GYH',
          tooltip: 'table.team.tooltip',
          sortable: true
        },
        p1: {
          label: 'VH',
          tooltip: 'table.team.tooltip',
          sortable: true,
          defaultSort: false
        },
        p0: {
          label: 'V',
          tooltip: 'table.team.tooltip',
          sortable: true,
          defaultSort: false
        },
        plus: {
          label: 'SZG',
          tooltip: 'table.team.tooltip',
          sortable: true
        },
        minus: {
          label: 'KG',
          tooltip: 'table.team.tooltip',
          sortable: true,
          defaultSort: false
        },
        gk: {
          label: 'GK',
          tooltip: 'table.team.tooltip',
          sortable: true
        },
        p: {
          label: 'P',
          tooltip: 'table.team.tooltip',
          sortable: true
        }
      },
      rows: [],
      SortService: null,
      sort: {
        sortTarget: 'p',
        sortReverse: false
      },
      isLoading: false
    };
  },

  computed: {
    convertedData() {
      return convert(this.rows)
        .sorted(this.sort)
        .addContinuousIndex()
        .value();
    }
  },

  created() {
    this.SortService = new SortService(this.columns, this.sort);
  },

  mounted() {
    this.getData();
    this.$i18n.locale = this.lang;
  },

  methods: {
    async getData() {
      try {
        this.isLoading = true;
        const response = await fetchVBRData('standings', {
          championshipId: Number(this.championshipId),
          division: this.division
        });
        this.isLoading = false;
        this.rows = response ? response : [];
      } catch (error) {
        this.error = error.message;
        this.isLoading = false;
      }
    },

    onSort(column) {
      this.sort = this.SortService.set(column).get();
    }
  }
};
</script>

<style src="../../assets/scss/main.scss" lang="scss"></style>
