<template>
    <div id="itemstyle">
            <li><h3>{{ item.title }}</h3></li>
            <li>{{ item.type }}</li>
            <li>{{ item.price }} {{ currentCurrency }}</li>
            <li>{{ item.animals }}</li>
            <li v-if="item.category">{{ item.category }}</li>
            <button type="button" @click="deleteItem()">USUŃ</button>
    </div>
</template>

<script>
export default {
    props: ['item', 'currentCurrency'],
    methods: {
        deleteItem() {
            this.$emit('delete-item', this.item.id);
        }
    }
}
</script>

<style>
#itemstyle {
    position:relative;
    padding:5px;
    background-color: rgb(58, 58, 58);
    width:100%;
    height:fit-content;
    border-radius:5px;
    margin-bottom:10px;
}
li {
    list-style-type:none;
    color:white;
}
li:nth-child(1) {
    float:left;
    display:block;
    margin:0 15px 0 5px;
}
#itemstyle button {
    font-size:15px;
    padding:5px;
    background-color:rgb(221, 101, 4);
    color:white;
    border:1px solid rgb(221, 101, 4);
    border-radius:5px;
    position:absolute;
    top:5px;
    right:5px;
    width:fit-content;
    height:fit-content;
    transition:0.1s;
}
#itemstyle button:hover {
    background-color:rgb(243, 110, 1);
    border:1px solid rgb(243, 110, 1);
}
</style>