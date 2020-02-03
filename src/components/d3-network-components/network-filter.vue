<template>
    <div class="net-menu">
        <el-input
                style="margin-bottom: 5px"
                placeholder="Search Names"
                size="small"
                prefix-icon="el-icon-search"
                v-model="searchKey"
                v-on:change="searchKeyFilterChange"></el-input>
        <el-table
                :stripe="false"
                :border="true"
                height="500"
                :fit="true"
                :data="searchedNodes"
                ref="filterTable"
                :row-class-name="accountsTableRowClassName"
                v-on:selection-change="handleFilterSelectionChange">
            <el-table-column
                    type="selection"
                    width="35">
            </el-table-column>
            <el-table-column
                    prop="name"
                    label="Name"
                    min-width="180">
            </el-table-column>
            <el-table-column
                    sortable
                    prop="data.weight.total"
                    label="Total">
                <template slot-scope="scope">
                    {{scope.row.data.weight.total }}
                </template>
            </el-table-column>
        </el-table>
        <el-dialog title="Advance Filter"
                   :visible.sync="advanceFilterModal.visible"
                   :modal="false"
                   :show-close="true"
                   :fullscreen="true">
            <el-card class="highlight-table">
                <el-row slot="header" type="flex" justify="start">
                    <el-col :span="4">
                        <el-input
                                placeholder="Search Emails And Names"
                                prefix-icon="el-icon-search"
                                size="small"
                                v-model="advanceFilterModal.searchKey"
                                v-on:change="advanceFilterFilterChange">
                        </el-input>
                    </el-col>
                </el-row>
                <el-table
                        :stripe="false"
                        :border="false"
                        :data="advanceFilterModal.filterNodes"
                        ref="advanceFilterTable"
                        :row-class-name="accountsTableRowClassName"
                        v-on:filter-change="advanceFilterFilterChange"
                        v-on:selection-change="handleAdvanceFilterSelectionChange"
                        max-height="650"
                        style="width: 100%">
                    <el-table-column
                            type="selection"
                            width="55">
                    </el-table-column>
                    <el-table-column
                            prop="data.public_name"
                            label="Name"
                            min-width="200">
                    </el-table-column>
                    <el-table-column
                            prop="data.email"
                            label="Email"
                            min-width="250">
                    </el-table-column>
                    <el-table-column
                            label="Category"
                            prop="data.categories"
                            width="150" :filters="advanceFilterModal.categoryFilter"
                            :filter-method="filterCategory" filter-placement="bottom-end">
                        <template slot-scope="account">
                            <el-tag type="primary" v-if="account.row.data.categories.length!==0"
                                    v-for="item in account.row.data.categories" :key="item.name">{{item.name}}
                            </el-tag>
                        </template>
                    </el-table-column>

                    <el-table-column
                            width="110" :filters="advanceFilterModal.tagsFilter"
                            prop="data.tags"
                            :filter-method="filterTags" filter-placement="bottom-end"
                            label="Tags">
                        <template slot-scope="account">
                            <el-tag type="success" v-for="tag in account.row.data.tags" :key="tag.id"
                                    style="margin-top: 2px;margin-bottom: 2px;">{{tag.name}}
                            </el-tag>
                        </template>
                    </el-table-column>
                </el-table>
            </el-card>
            <div slot="footer">
                <el-button type="primary" icon="el-icon-check" v-on:click="advanceFilter" size="small">
                    Choose
                </el-button>
                <el-button type="danger" icon="el-icon-close" :plain="true"
                           v-on:click="advanceFilterModal.visible = false;"
                           size="small">
                    Close
                </el-button>
            </div>
        </el-dialog>

        <el-row type="flex" justify="space-between" align="middle" style="margin-top: 10px;">
            <el-col :span="6">
                <el-button type="text" v-on:click="showAdvanceFilterModal">
                    Advance Filter
                </el-button>
            </el-col>
            <el-col :span="4">
                <el-button type="primary" v-on:click="filter" size="small">
                    <icon name="check">
                    </icon>
                    Filter
                </el-button>
            </el-col>

        </el-row>
    </div>
</template>
<script>
    export default {
        name: 'NetworkFilter',
        props: ['nodes', 'common', 'excluded', 'doLayout'],
        data() {
            return {
                allNodes: [],
                graphNodes: [],
                clearSelection: false,
                selectedNodes: [],
                filteredNodes: [],
                searchedNodes: [],
                searchKey: '',
                advanceFilterModal: {
                    visible: false,
                    selection: [],
                    filterOptions: {},
                    categoryFilter: [],
                    tagsFilter: [],
                    filterNodes: [],
                    searchKey: ''
                }
            };
        },
        created() {
        },
        computed: {
            networkMiner() {
                return vuex.getters.networkMiner
            }
        },
        mounted() {
            this.allNodes = this.nodes.concat(this.excluded);
            this.selectedNodes = this.nodes;
            vuex.dispatch('setNetworkMiner', {selectedNodes: this.nodes, allNodes: this.nodes.concat(this.excluded)});
            this.searchKeyFilterChange();
            this.advanceFilterModal.filterOptions.categories = [];
            this.advanceFilterModal.filterOptions.groups = [];
            this.advanceFilterModal.filterOptions.tags = [];
        },
        watch: {
            'nodes': {
                handler(val, oldVal) {
                    this.filteredNodes = $(this.networkMiner.allNodes).not(this.networkMiner.selectedNodes).get()
                    this.graphNodes = val;
                    this.allNodes = this.graphNodes.concat(this.filteredNodes);
                    vuex.dispatch('setNetworkMiner', {
                        allNodes: this.allNodes,
                        selectedNodes: $(this.allNodes).not(this.filteredNodes).get()
                    });
                    //extend
                    this.selectedNodes = this.networkMiner.selectedNodes;
                    this.searchKeyFilterChange();
                    if (this.$refs.advanceFilterTable) {
                        this.clearSelection = true;
                        this.$refs.advanceFilterTable.clearSelection();
                        this.clearSelection = false;

                    }
                }
            },
            'excluded': {
                handler(val, oldVal) {
                    this.filteredNodes = val;
                    this.allNodes = this.graphNodes.concat(val);
                    vuex.dispatch('setNetworkMiner', {allNodes: this.allNodes});
                    //extend
                    this.selectedNodes = this.networkMiner.selectedNodes;
                    this.searchKeyFilterChange();
                    if (this.$refs.advanceFilterTable) {
                        this.clearSelection = true;
                        this.$refs.advanceFilterTable.clearSelection();
                        this.clearSelection = false;
                    }
                }
            },
            'doLayout': {
                handler(val, oldVal) {
                    this.filterTableDoLayout();
                }
            }
        },
        methods: {
            /**
             * make an array unique
             *
             * @param {array} array
             * @param {string} type
             * @return {array}
             */
            arrayUnique(array, type = 'id') {
                let a = array.concat();
                for (let i = 0; i < a.length; ++i) {
                    for (let j = i + 1; j < a.length; ++j) {
                        if (type === 'id') {
                            if (a[i].id === a[j].id) {
                                a.splice(j--, 1);
                            }
                        } else if (type === 'tid-sid') {
                            if (a[i].tid === a[j].tid && a[i].sid === a[j].sid) {
                                a.splice(j--, 1);
                            }
                        } else {
                            if (a[i] === a[j]) {
                                a.splice(j--, 1);
                            }
                        }
                    }
                }

                return a;
            },
            /**
             * filter nodes
             *
             * @return {void}
             */
            filter() {
                let selectedIds = this.networkMiner.selectedNodes.map((val) => {
                    return val.id;
                });
                let allIds = this.networkMiner.allNodes.map((val) => {
                    return val.id;
                });
                let filteredIds = $(allIds).not(selectedIds).get();
                this.emit({excluded: filteredIds});
            },
            /**
             * advance filter nodes
             *
             * @return {void}
             */
            advanceFilter() {
                this.selectFilteredNodes(this.networkMiner.selectedNodes);
                this.advanceFilterModal.visible = false;
            },
            /**
             * show advance filter modal
             *
             * @return {void}
             */
            showAdvanceFilterModal() {
                this.advanceFilterModal.searchKey = '';
                this.advanceFilterModal.filterNodes = this.networkMiner.allNodes;
                this.advanceFilterModal.visible = true;
                let tempSelectedNodes=this.networkMiner.selectedNodes;

                this.$nextTick().then(() => {
                    this.clearSelection = true;
                    this.$refs.advanceFilterTable.clearSelection();
                    this.clearSelection = false;
                    vuex.dispatch('setNetworkMiner', {selectedNodes: tempSelectedNodes});
                    setTimeout(() => {
                        $.each(this.networkMiner.selectedNodes, (key, val) => {
                            this.$refs.advanceFilterTable.toggleRowSelection(val);
                        });
                    });
                });
                axios.get(laroute.route('accounts.filter_options_data')).then((res) => {
                    this.advanceFilterModal.tagsFilter = [];
                    $.each(res.data.tags, (key, val) => {
                        this.advanceFilterModal.tagsFilter.push({text: val.label, value: val.value});
                    });
                    this.advanceFilterModal.tagsFilter.push({text: 'not defined', value: -1});

                    this.advanceFilterModal.categoryFilter = [];

                    let stack = res.data.categories;
                    while (stack.length !== 0) {
                        let node = stack.pop();
                        if (node.children.length === 0) {
                            let label = null;
                            if (node.text) {
                                label = node.text + node.name
                            } else {
                                label = node.name
                            }
                            let newArray = {
                                "text": label,
                                "value": node.id
                            };
                            this.advanceFilterModal.categoryFilter.push(newArray);

                        } else {
                            let label = null;
                            if (node['text']) {
                                label = node['text'] + node.name
                            } else {
                                label = node.name
                            }
                            let newArray = {
                                "text": label,
                                "value": node.id
                            };
                            this.advanceFilterModal.categoryFilter.push(newArray);
                            for (let i = node.children.length - 1; i >= 0; i--) {
                                if (!node['text']) {
                                    node['text'] = '';
                                }
                                node.children[i]['text'] = node.text + node.name + "/";
                                stack.push(node.children[i]);
                            }
                        }
                    }

                    this.advanceFilterModal.categoryFilter.push({text: 'not defined', value: -1});
                }).catch((error) => {
                    app.errorHandler(error);
                    this.loading = false;
                });
            },
            /**
             * handle filter selection change
             *
             * @param {object} val
             * @return {void}
             */
            handleFilterSelectionChange(val) {
                let selectedSearchNodeIds = val.map((val) => {
                    return val.id;
                });
                let filteredSearchNodes = this.searchedNodes.filter((val) => {
                    return !selectedSearchNodeIds.includes(val.id);
                });
                let filteredSearchNodeIds = filteredSearchNodes.map((val) => {
                    return val.id;
                });
                let selectedNodeIds = this.networkMiner.selectedNodes.map((val) => {
                    return val.id;
                });
                let selectedNodes = this.networkMiner.allNodes.filter((val) => {
                    return ((selectedSearchNodeIds.includes(val.id) || selectedNodeIds.includes(val.id)) && !filteredSearchNodeIds.includes(val.id));
                });
                vuex.dispatch('setNetworkMiner', {selectedNodes: selectedNodes});
            },
            /**
             * handle change in advance filter selection
             *
             * @param {object} val
             * @return {void}
             */
            handleAdvanceFilterSelectionChange(val) {

                if (!this.clearSelection ) {
                    let selectedSearchNodeIds = val.map((val) => {
                        return val.id;
                    });
                    let filteredSearchNodes = this.advanceFilterModal.filterNodes.filter((val) => {
                        return !selectedSearchNodeIds.includes(val.id);
                    });
                    let filteredSearchNodeIds = filteredSearchNodes.map((val) => {
                        return val.id;
                    });
                    let selectedNodeIds = this.networkMiner.selectedNodes.map((val) => {
                        return val.id;
                    });
                    let selectedNodes = this.networkMiner.allNodes.filter((val) => {
                        return ((selectedSearchNodeIds.includes(val.id) || selectedNodeIds.includes(val.id)) && !filteredSearchNodeIds.includes(val.id));
                    });
                    vuex.dispatch('setNetworkMiner', {selectedNodes: selectedNodes});


                }
            },
            /**
             * select nodes in filterTable
             *
             * @param {object} selected
             * @return {void}
             */
            selectFilteredNodes(selected) {
                this.clearSelection = true;
                this.$refs.filterTable.clearSelection();
                this.$nextTick().then(() => {
                    let selectedIds = selected.map((val) => {
                        return val.id;
                    });
                    let selectedRows = this.searchedNodes.filter((val) => {
                        return selectedIds.includes(val.id);
                    });
                    selectedRows.forEach(row => {
                        this.$refs.filterTable.toggleRowSelection(row);
                    });
                });
                this.clearSelection = false;

            },
            /**
             * filter category method
             *
             * @param value
             * @param row
             * @return {boolean}
             */
            filterCategory(value, row) {
                return true;
            },
            /**
             * filter group method
             *
             * @param value
             * @param row
             * @return {boolean}
             */
            filterGroup(value, row) {
                return true;
            },
            /**
             * filter tags method
             *
             * @param value
             * @param row
             * @return {boolean}
             */
            filterTags(value, row) {
                return true;
            },
            /**
             * filter status method
             *
             * @param value
             * @param row
             * @return {boolean}
             */
            filterStatus(value, row) {
                return true;
            },
            /**
             * method to handle filter changes
             *
             * @param {object} filters
             * @return {void}
             */
            advanceFilterFilterChange(filters = null) {
                if (filters) {
                    let filterCol = $('.' + Object.keys(filters)[0]).eq(0).children('div.cell').text();
                    let filterObjName = Object.keys(filters)[0];
                    //category
                    if (filterCol === 'Category') {
                        this.$set(this.advanceFilterModal.filterOptions, 'categories', filters[filterObjName]);
                    }
                    //groups
                    else if (filterCol === 'Groups') {
                        this.$set(this.advanceFilterModal.filterOptions, 'groups', filters[filterObjName]);
                    }
                    //tags
                    else if (filterCol === 'Tags') {
                        this.$set(this.advanceFilterModal.filterOptions, 'tags', filters[filterObjName]);
                    }
                }
                this.advanceFilterModal.filterNodes = this.allNodes.filter((el) => {
                        let currentCategoryId = [];
                        let currentGroupsId = [];
                        if (el.data.categories) {

                            el.data.categories.forEach((val, key) => {
                                currentCategoryId.push(val.id);
                            });

                        }
                        let currentTagsId = el.data.tags.map((val) => {
                            return val.id;
                        });
                        if ((!this.advanceFilterModal.filterOptions.categories || this.advanceFilterModal.filterOptions.categories.length === 0) && this.advanceFilterModal.filterOptions.groups.length === 0 && this.advanceFilterModal.filterOptions.tags.length === 0 && this.advanceFilterModal.searchKey === '') {
                            return true;
                        }
                        let categoryExist = false;

                        this.advanceFilterModal.filterOptions.categories.forEach((val, key) => {
                            currentCategoryId.forEach((value, keys) => {
                                if (val === value) {
                                    categoryExist = true;
                                }

                            });
                            if ((val === -1 && (!el.data.categories || el.data.categories.length === 0)) || this.advanceFilterModal.filterOptions.categories.length === 0) {
                                categoryExist = true;
                            }
                        });


                        if (((this.advanceFilterModal.filterOptions.categories.includes(-1)) && (!el.data.categories || el.data.categories.length === 0)) || this.advanceFilterModal.filterOptions.categories.length === 0) {
                            categoryExist = true;
                        }
                        let groupExist = false;
                        $.each(currentGroupsId, (key, val) => {
                            if (this.advanceFilterModal.filterOptions.groups.includes(val)) {
                                groupExist = true;
                            }
                        });
                        if (((this.advanceFilterModal.filterOptions.groups.includes(-1)) && (!el.data.category || el.data.category.length === 0 || !el.data.category.groups || el.data.category.groups.length === 0)) || this.advanceFilterModal.filterOptions.groups.length === 0) {
                            groupExist = true;
                        }
                        let tagExist = false;

                        $.each(currentTagsId, (key, val) => {
                            if (this.advanceFilterModal.filterOptions.tags.includes(val)) {

                                tagExist = true;
                            }
                            if ((val === -1 && (!el.data.tags || el.data.tags.length === 0)) || this.advanceFilterModal.filterOptions.tags.length === 0) {
                                tagExist = true;
                            }
                        });

                        if (((this.advanceFilterModal.filterOptions.tags.includes(-1)) && (!el.data.tags || el.data.tags.length === 0)) || this.advanceFilterModal.filterOptions.tags.length === 0) {
                            tagExist = true;
                        }
                        let searchKeyExist = true;
                        if (el.data && el.data.email && el.data.email.toLowerCase().indexOf(this.advanceFilterModal.searchKey.toLowerCase()) === -1 && el.name && el.name.toLowerCase().indexOf(this.advanceFilterModal.searchKey.toLowerCase()) === -1 && this.advanceFilterModal.searchKey !== '') {
                            searchKeyExist = false;
                        }
                        return categoryExist && groupExist && tagExist && searchKeyExist;
                    }
                );

                this.clearSelection = true;
                let lastSelected = this.networkMiner.selectedNodes
                this.$refs.advanceFilterTable.clearSelection();
                this.clearSelection = false;

                this.$nextTick().then(() => {
                    let selectedIds = lastSelected.map((val) => {
                        return val.id;
                    });

                    let selectedRows = this.advanceFilterModal.filterNodes.filter((val) => {
                        return selectedIds.includes(val.id);
                    });
                    selectedRows.forEach(row => {
                        this.$refs.advanceFilterTable.toggleRowSelection(row);
                    });
                });
            },
            /**
             * fires when searchKey Filter Change
             *
             * @return {void}
             */
            searchKeyFilterChange() {
                this.searchedNodes = this.networkMiner.allNodes.filter((el) => {
                    let searchKeyExist = true;
                    if (el.name && el.name.toLowerCase().indexOf(this.searchKey.toLowerCase()) === -1 && this.searchKey !== '') {
                        searchKeyExist = false;
                    }
                    return searchKeyExist;
                });
                this.selectFilteredNodes(this.networkMiner.selectedNodes);
            },
            /**
             * emit action to parent component
             *
             * @param {array|object} args
             * @return {void}
             */
            emit(args) {
                this.$emit('action', args);
            },
            /**
             *  account table row class
             *
             * @param {object} row
             * @param {number} rowIndex
             * @return {string}
             */
            accountsTableRowClassName({row, rowIndex}) {
                if (vuex.getters.minerProperties.accounts.includes(row.id)) {
                    return 'success-row';
                }
                return '';
            },
            /**
             * do layout table
             *
             * @return {void}
             */
            filterTableDoLayout() {
                this.$refs.filterTable.doLayout();
            }
        }
    }
</script>
<style scoped>
    .debug {
        font-size: 0.5em;
        list-style: none;
    }

    .net-menu {
        padding: 1em;
        list-style: none;
    }

    .net-menu li {
        margin: 0.5em 0;
    }

    .net-menu li label {
        font-size: 0.85em;
        display: block;
    }

    .net-menu li label span {
        font-weight: normal;
    }

    .net-menu li input + label {
        display: inline;
        margin-left: 0.5em;
    }

    ul.net-menu + ul.net-menu {
        border-left: none;
    }

</style>
