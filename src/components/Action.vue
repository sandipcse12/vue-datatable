<template>
    <div>
        <a href="javascript:" @click="edit">Edit</a>
        <a href="javascript:" @click="edit" style="padding-left: 10px; color:red">Delete</a>
    </div>
</template>
<script>
    export default {
        props: ['row'],
        name: "Action",
        data(){
            return{

            }
        },
        created() {
        },
        methods:{
            edit(){
                console.log(this.row.id);
            }
        }
    }
</script>

