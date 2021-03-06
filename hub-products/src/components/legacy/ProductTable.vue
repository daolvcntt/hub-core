<template>
    <div>
        <search-table includes="channels,customer_groups,family,variants,assets.transforms" type="product" export="products">
            <template slot="cols">
                <th width="5%"></th>
                <th width="25%">Product</th>
                <th width="10%">Stock</th>
                <th width="15%">Channels</th>
                <th width="19%">Customer Groups</th>
                <th width="19%">Purchasable</th>
            </template>
            <template slot-scope="products">
                <tr :key="product.id" v-for="(product, index) in products.default">
                    <td>
                        <nuxt-link :to="{
                            name: 'products.view',
                            params: {
                                id: product.id
                            }
                        }">
                            <thumbnail-loader :item="product"></thumbnail-loader>
                        </nuxt-link>
                    </td>
                    <td width="70%">
                        <nuxt-link :to="{
                            name: 'products.view',
                            params: {
                                id: product.id
                            }
                        }">
                            {{ product.attribute_data['name'][channel][locale] }}
                        </nuxt-link>
                    </td>
                    <td>
                        <template v-if="editing == product && product.variants.data.length == 1 && !editModal">
                            <v-text-field v-focus @keyup.enter="quickSave" class="form-control" v-model="product.variants.data[0].inventory" @blur="quickSave" />
                        </template>
                        <template v-else>
                            <a href="#" data-toggle="tooltip" v-on:click.prevent="quickEdit(product)" data-placement="top" title="Edit" class="editable-stock">
                                {{ getStock(product) }}
                            </a>
                        </template>
                    </td>
                    <td>{{ visibility(product, 'channels') }}</td>
                    <td>{{ visibility(product, 'customer_groups') }}</td>
                    <td>{{ purchasable(product, 'customer_groups') }}</td>
                </tr>
            </template>
        </search-table>

        <v-dialog title="Edit Stock" v-show="editModal" size="modal-md" @closed="editModal = false" v-if="editing">
            <div slot="body">
                <div v-for="variant in editing.variants.data" class="form-group" :key="variant.id">
                    <label>{{ variant.sku }}</label>
                    <input class="form-control" v-model="variant.inventory">
                </div>
            </div>
            <template slot="footer">
                <button type="button" class="btn btn-primary" @click="quickSave">Save Stock</button>
            </template>
        </v-dialog>
    </div>
</template>

<script>
    import SearchTable from './SearchTable.vue';
    import ThumbnailLoader from '../ThumbnailLoader.vue';

    import HasGroups from '../../mixins/HasGroups.js';
    // import HasAttributes from '../../../mixins/HasAttributes.js';

    export default {
        components: {
            SearchTable,
            ThumbnailLoader
        },
        mixins: [
            HasGroups,
            // HasAttributes,
        ],
        directives: {
            focus: {
                // directive definition
                inserted: function (el) {
                    el.focus()
                }
            }
        },
        data() {
            return {
                editing: null,
                editModal: false,
            }
        },
        methods: {
            getStock(product) {
                var variants = product.variants.data;
                if (variants.length == 1) {
                    return variants[0].inventory;
                }
                return 'Multiple';
            },
            quickEdit(index) {
                this.editing = index;
                this.editingBackup = JSON.parse(JSON.stringify(this.editing));
                if (this.editing.variants.data.length > 1) {
                    this.editModal = true;
                }
            },
            quickSave() {
                var product = this.editing;
                var variants = product.variants.data;

                variants.forEach((variant, index) => {
                    // Only save if it has changed.
                    if (JSON.stringify(variant) == JSON.stringify(this.editingBackup.variants.data[index])) {
                        return;
                    }
                    const apiRequest = this.$nuxt.context.app.$axios;

                    apiRequest.put('/products/variants/' + variant.id + '/inventory', {
                        inventory: variant.inventory
                    })
                        .then(response => {
                            CandyEvent.$emit('notification', {
                                level: 'success',
                                message: 'Stock Updated'
                            });
                        }).catch(response => {
                            // CandyEvent.$emit('notification', {
                            //     level: 'error',
                            //     message: response.message
                            // });
                            return false;
                        });
                });

                this.editingBackup = null;
                this.editing = null;
                this.editModal = false;
            },
            cancelQuickEdit() {
                this.products[this.editing] = this.editingBackup;
                this.editingBackup = null;
                this.editing = null;
            },
        },
        computed: {
            locale() {
                return this.$store.state.core.locale;
            },
            channel() {
                return this.$store.state.core.channel;
            }
        }
    }
</script>

<style lang="scss" scoped>
    .editable-stock {
        padding:2px 4px;
        position: relative;
        border:1px dashed transparent;
        color:#333;
        &:hover {
            text-decoration: none;
            background-color:#f5f5f5;
            border-color: #BEBEBE;
        }
    }
</style>
