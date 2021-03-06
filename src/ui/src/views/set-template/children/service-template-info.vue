<template>
    <section>
        <p class="title">{{`${$t('服务分类')}：${serviceCategory}`}}</p>
        <bk-table ref="table"
            v-bkloading="{ isLoading: $loading() }"
            :data="processes"
            :show-header="!!processes.length"
            :height="276"
            :max-height="276">
            <bk-table-column v-for="head in header"
                :key="head.id"
                :prop="head.id"
                :label="head.name">
                <template slot-scope="{ row }">{{row[head.id] | formatter(head.property)}}</template>
            </bk-table-column>
        </bk-table>
    </section>
</template>

<script>
    export default {
        name: 'serviceTemplateInfo',
        props: {
            id: {
                type: Number,
                required: true
            }
        },
        data () {
            return {
                serviceCategory: '',
                processes: [],
                properties: []
            }
        },
        computed: {
            header () {
                const display = [
                    'bk_func_name',
                    'bk_process_name',
                    'bk_start_param_regex',
                    'bind_ip',
                    'port',
                    'protocol',
                    'work_path',
                    'user'
                ]
                const header = display.map(id => {
                    const property = this.properties.find(property => property.bk_property_id === id) || {}
                    return {
                        id: property.bk_property_id,
                        name: this.$tools.getHeaderPropertyName(property),
                        property
                    }
                })
                return header
            }
        },
        async created () {
            setTimeout(() => {
                this.$refs.table.doLayout()
            }, 0)
            this.getProcessProperties()
            this.getTitle()
            this.getServiceProcesses()
        },
        methods: {
            close () {
                this.visible = false
            },
            async getProcessProperties () {
                try {
                    const action = 'objectModelProperty/searchObjectAttribute'
                    this.properties = await this.$store.dispatch(action, {
                        params: {
                            bk_obj_id: 'process',
                            bk_supplier_account: this.$store.getters.supplierAccount
                        },
                        config: {
                            requestId: 'get_service_process_properties',
                            fromCache: true
                        }
                    })
                } catch (e) {
                    console.error(e)
                }
            },
            async getServiceProcesses () {
                try {
                    const result = await this.$store.dispatch('processTemplate/getBatchProcessTemplate', {
                        params: this.$injectMetadata({
                            service_template_id: this.id
                        }),
                        config: {
                            requestId: 'getServiceProcesses'
                        }
                    })
                    this.processes = result.info.map(data => {
                        const process = {}
                        Object.keys(data.property).forEach(key => {
                            process[key] = data.property[key].value
                        })
                        return process
                    })
                } catch (e) {
                    console.error(e)
                    this.processes = []
                }
            },
            async getTitle () {
                try {
                    const [details, categoryData] = await Promise.all([
                        this.getServiceDetails(),
                        this.getServiceCategory()
                    ])
                    const categories = categoryData.info
                    const categoryId = details.template.service_category_id
                    const subCategory = categories.find(data => data.id === categoryId) || {}
                    const category = categories.find(data => data.id === subCategory.bk_parent_id) || {}
                    this.serviceCategory = `${category.name} / ${subCategory.name}`
                } catch (e) {
                    console.error(e)
                    this.serviceCategory = ''
                }
            },
            getServiceDetails () {
                return this.$store.dispatch('serviceTemplate/findServiceTemplate', {
                    id: this.id,
                    config: {
                        requestId: 'getServiceDetails'
                    }
                })
            },
            getServiceCategory () {
                return this.$store.dispatch('serviceClassification/searchServiceCategoryWithoutAmout', {
                    params: this.$injectMetadata({}),
                    config: {
                        requestId: 'getServiceCategoryWithoutAmount'
                    }
                })
            }
        }
    }
</script>

<style lang="scss" scoped>
    .title {
        font-size: 14px;
        line-height: 20px;
        margin-top: -6px;
        padding: 0 0 16px 0;
    }
</style>
