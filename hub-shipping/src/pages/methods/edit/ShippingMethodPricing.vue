<template>
  <div>
    <div v-for="(rates, index) in groupPricing(prices)" :key="index">
      <header class="px-6 py-3 bg-white border-t border-b shadow-sm">
        <h2 class="font-medium">{{ rates[0].zone.data.name }}</h2>
      </header>

          <gc-table
            :data="rates"
            :columns="[
              {label: $t('Rate'), field: 'rate'},
              {label: $t('Currency'), field: 'currency'},
              {label: $t('Min Basket'), field: 'min_basket'},
              {label: $t('Min Weight'), field: 'min_weight'},
              {label: $t('Min Height'), field: 'min_height'},
              {label: $t('Min Width'), field: 'min_width'},
              {label: $t('Min Depth'), field: 'min_depth'},
              {label: $t('Min Volume'), field: 'min_volume'},
              {label: '', field: 'actions'},
            ]"
          >
            <template v-slot:rate="{ row }">
              {{ $format.currency(row.rate, row.currency.data) }}
            </template>
            <template v-slot:currency="{ row }">
              {{ row.currency.data.name }}
            </template>
            <template v-slot:min_basket="{ row }">
              {{ $format.currency(row.min_basket, row.currency.data) }}
            </template>
            <template v-slot:min_weight="{ row }">
              {{ row.min_weight }}{{ row.weight_unit }}
            </template>
            <template v-slot:min_height="{ row }">
              {{ row.min_height }}{{ row.height_unit }}
            </template>
            <template v-slot:min_width="{ row }">
              {{ row.min_width }}{{ row.width_unit }}
            </template>
            <template v-slot:min_depth="{ row }">
              {{ row.min_depth }}{{ row.depth_unit }}
            </template>
            <template v-slot:min_volume="{ row }">
              {{ row.min_volume }}{{ row.volume_unit }}
            </template>
            <template v-slot:actions="{ row }">
                <gc-button  @click="editing = row" size="x-small">
                  <b-icon icon="edit-line" />
                </gc-button>
                <gc-button @click="showDeleteConfirm = true" theme="danger" size="x-small">
                  <b-icon icon="delete-bin-line" />
                </gc-button>
              </template>
          </gc-table>
        <!-- <b-table
          :data="rates"
          paginated
        >
          <template v-slot="props">
            <b-table-column field="rate" :label="$t('Rate')" sortable>
              {{ $format.currency(props.row.rate, props.row.currency.data) }}
            </b-table-column>
            <b-table-column field="currency" :label="$t('Currency')">
              {{ props.row.currency.data.name }}
            </b-table-column>
            <b-table-column field="min_basket" :label="$t('Min Basket')">
              {{ $format.currency(props.row.min_basket, props.row.currency.data) }}
            </b-table-column>
            <b-table-column field="min_weight" :label="$t('Min Weight')">
              {{ props.row.min_weight }}
              {{ props.row.weight_unit }}
            </b-table-column>
            <b-table-column field="min_height" :label="$t('Min Height')">
              {{ props.row.min_height }}
              {{ props.row.height_unit }}
            </b-table-column>
            <b-table-column field="min_width" :label="$t('Min Width')">
              {{ props.row.min_width }}
              {{ props.row.width_unit }}
            </b-table-column>
            <b-table-column field="min_depth" :label="$t('Min Depth')">
              {{ props.row.min_depth }}
              {{ props.row.depth_unit }}
            </b-table-column>
            <b-table-column field="min_volume" :label="$t('Min Volume')">
              {{ props.row.min_volume }}
              {{ props.row.volume_unit }}
            </b-table-column>
            <b-table-column field="actions">
              <template>
                <button  @click="editing = props.row" class="inline-flex justify-center px-1 py-1 text-base font-medium leading-6 text-gray-600 transition duration-150 ease-in-out bg-white border border-gray-300 rounded-md shadow-sm hover:text-gray-500 focus:outline-none focus:border-blue-300 focus:shadow-outline sm:text-sm sm:leading-5">
                  <b-icon icon="edit-line" />
                </button>
                <button @click="showDeleteConfirm = true" class="inline-flex justify-center px-1 py-1 text-base font-medium leading-6 text-white transition duration-150 ease-in-out bg-red-600 border border-transparent rounded-md shadow-sm hover:bg-red-400 focus:outline-none focus:border-blue-300 focus:shadow-outline sm:text-sm sm:leading-5">
                  <b-icon icon="delete-bin-line" />
                </button>
              </template>

            </b-table-column>
          </template>
        </b-table> -->
    </div>
    <quick-view-panel :open="true" :heading="$t('Edit price')" v-if="editing" @close="editing = null" :takeover="true">
      <shipping-method-price-form :existing="editing" />
    </quick-view-panel>
  </div>
</template>

<script>
  const groupBy = require('lodash/groupBy')

  import ShippingMethodPriceForm from '../../../components/ShippingMethodPriceForm.vue'
  import NormalizesObjects from '@getcandy/hub-core/src/mixins/NormalizesObjects.js'

  export default {
    layout: 'shipping-method',
    mixins: [
      NormalizesObjects
    ],
    components: {
      ShippingMethodPriceForm
    },
    data() {
      return {
        editing: null,
        prices: []
      }
    },
    methods: {
      groupPricing(prices) {
        return groupBy(prices, (price) => {
          return price.zone.data.id;
        })
      }
    },
    mounted () {
      this.prices = this.normalize(this.model.prices.data)
    },
    computed: {
      model () {
        return this.$store.state.shippingMethods.model
      },
      currencies () {
        return this.$store.state.core.currencies
      }
    }
  }
</script>

<style scoped>

</style>
