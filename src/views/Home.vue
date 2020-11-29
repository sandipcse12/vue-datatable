<template>
    <div class="home">
        <Datatable :options="tableOptions"/>
    </div>
</template>

<script>
    // @ is an alias to /src
    import Datatable from '@/components/Datatable.vue'

    export default {
        name: 'Home',
        components: {
            Datatable
        },
        data() {
            return {
                tableOptions : {
                    tableName: 'users',
                    columns: [
                        {title: 'Name', key: 'name', type: 'clickable',source:'users', uniqueField:'id', sortable: true},
                        {title: 'Age', key: 'age', type: 'text', sortable: true},
                        {title: 'Salary', key: 'salary', type: 'text', sortable: false},
                        // {title: 'Action',  type: 'component', name:'home-action'},

                    ],
                    source: 'users',
                    search: true,
                },
            }
        }
    }
</script>
<style lang="scss" scoped>
    .home{
        margin:20px;
    }
</style>
