<template>
    <div>
        <div class="search">
            <input type="text" style="float:right" placeholder="Search here" id="search" @keyup="search" v-model="requestParams.search">
            Show <select class="form-control" v-model="showDataAmount" @change="selectedDataAmount" id="rows">
                <option value="10">10</option>
                <option value="20">20</option>
                <option value="50">50</option>
                <option value="100">100</option>
                <option value="500">500</option>
            </select> rows
        </div>
        <table id="datatable">
            <thead>
            <th v-for="(column,columnIndex) in options.columns" :style="{'text-align':column.textAlign}" :key="columnIndex">
                <span>{{ column.title }}</span>
                <span style="float: right"
                      v-if="requestParams.sortedKey === column.key && requestParams.sortedType === 'asc'">
                                    <a href="javascript:" @click="sortedKeyValue(column.key,'desc')">
                                        <img src="../assets/up.png" alt="up.png">
                                    </a></span>
                <span style="float: right"
                      v-else-if="requestParams.sortedKey === column.key && requestParams.sortedType === 'desc'">
                                    <a href="javascript:" @click="sortedKeyValue(column.key,'asc')">
                                        <img src="../assets/down.png" alt="down.png">
                                    </a></span>
                <span style="float: right" v-else-if="column.sortable">
                                     <a href="javascript:" @click="sortedKeyValue(column.key,'desc')">
                                   <img src="../assets/sort.png" alt="sort.png">
                                     </a>
                                </span>
            </th>
            </thead>
            <tbody>
            <template v-if="dataSets.length > 0">
                <tr v-for="(data,dataIndex) in dataSets" :key="dataIndex">
                    <td v-for="(column,columnIndex) in options.columns" :key="columnIndex" :style="{'text-align':column.textAlign}" >
                                        <span v-if="column.type === 'component'">
                                            <component :is="column.name" :row="data"></component>
                                        </span>
                        <span v-else-if="column.type === 'clickable'">
                                            <a style="text-decoration: none"
                                               :href="column.source+'/'+data[column.uniqueField]">{{data[column.key]}}</a>
                                        </span>
                        <span v-else-if="column.currencyFormat">
                                              {{currencyFormat(data[column.key],column.currencyConfig)}}
                                        </span>
                        <span v-else-if="column.dateFormat">
                                              {{dateFormat(data[column.key],column.currentFormat,column.expectFormat)}}
                                        </span>
                        <span v-else>
                                              {{data[column.key]}}
                                        </span>

                    </td>
                </tr>
            </template>
            <template v-else>
                No data found.
            </template>
            </tbody>
        </table>

        <div class="pagination" v-if="totalCount > requestParams.take">
            <a href="javascript:" :class="{'isDisabled':currentPage==1}" @click="newPage(currentPage-1)">&laquo;</a>
            <a href="javascript:" @click="newPage(page)" :class="{'active':currentPage==page}"
               v-for="(page,pageIndex) in pages"
               :key="pageIndex" :disables="currentPage ===page || page === '...'">{{page}}</a>
            <a @click="newPage(currentPage+1)" :class="{'isDisabled':currentPage==totalPage}"
               href="javascript:">&raquo;</a>
        </div>
    </div>
</template>

<script>
    const axios = require('axios');
    const formatCurrency = require('format-currency');
    import moment from 'moment';

    export default {
        props: ['options'],
        data() {
            return {
                dataSets: [],
                requestParams: {
                    take: 10,
                    skip: 0,
                    search: '',
                    sortedType: 'asc',
                },
                currentPage: 1,
                pages: [],
                showDataAmount: 10,
                totalPage: 1,
                totalCount: 0
            }
        },
        watch: {},
        mounted() {
            this.requestParams.sortedKey = this.options.columns[0].key;
            this.readData();
        },
        methods: {
            currencyFormat(value, config) {
                return formatCurrency(value, config);
            },
            dateFormat(date, currentConfig, expectedConfig) {
                return moment(date, currentConfig).format(expectedConfig);
            },
            sortedKeyValue(key, type) {
                this.requestParams.sortedKey = key;
                this.requestParams.sortedType = type;
                this.readData();
            },
            readData() {
                let instance = this;
                axios.post(this.options.source, this.requestParams)
                    .then(function (response) {
                        instance.dataSets = response.data.data;
                        instance.totalCount = response.data.count;
                        instance.totalPage = Math.ceil(instance.totalCount / instance.requestParams.take);
                        instance.pages = instance.pagination(instance.currentPage, instance.totalPage);
                    })
                    .catch(function (err) {
                        console.log(err);
                    });
            },
            search() {
                this.currentPage = 1;
                this.readData();
            },
            selectedDataAmount() {
                this.requestParams.take = this.showDataAmount;
                this.readData();
            },
            newPage(page) {
                if (page != 0 && page <= this.totalPage) {
                    this.requestParams.skip = (page - 1) * this.requestParams.take;
                    this.currentPage = page;
                    this.readData();
                }
            },
            pagination(c, m) {
                var delta = 2,
                    range = [],
                    rangeWithDots = [],
                    l;

                range.push(1);
                for (let i = c - delta; i <= c + delta; i++) {
                    if (i < m && i > 1) {
                        range.push(i);
                    }
                }
                range.push(m);

                for (let i of range) {
                    if (l) {
                        if (i - l === 2) {
                            rangeWithDots.push(l + 1);
                        } else if (i - l !== 1) {
                            rangeWithDots.push('...');
                        }
                    }
                    rangeWithDots.push(i);
                    l = i;
                }

                return rangeWithDots;
            }

        }
    }
</script>
<style lang="scss" scoped>
    select,input[type=text] {
       // width: 100%;
        padding: 6px 12px;
        margin: 8px 0;
        border: 1px solid #f1f1f1;
        box-sizing: border-box;
    }
    input:focus{
        border: 1px solid #f1f1f1 !important;
    }
    #datatable {
        font-family: Arial, Helvetica, sans-serif;
        border-collapse: collapse;
        width: 100%;
    }

    #datatable td, #datatable th {
        border: 1px solid #ddd;
        padding: 8px;
    }

    #datatable tr:hover {
        background-color: #f1f1f1;
    }

    #datatable th {
        padding-top: 12px;
        padding-bottom: 12px;
        text-align: left;
        background: #f1f1f1;
    }

    .pagination {
        display: inline-block;
        margin-top: 12px;
        float: right;
    }

    .pagination a {
        color: black;
        float: left;
        padding: 8px 16px;
        text-decoration: none;
        transition: background-color .3s;
        border: 1px solid #ddd;
    }

    .pagination a.active {
        background-color: #f1f1f1;
        //color: white;
        // border: 1px solid #4CAF50;
    }

    .pagination a:hover:not(.active) {
        background-color: #ddd;
    }

    .isDisabled {
        cursor: not-allowed;
        opacity: 0.5;
    }
</style>
