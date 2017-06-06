<template>
    <div class="table">
        <div class="list" v-show="isShow === 'list'">
            <div class="filter">
                <form action="" name="searchForm">
                    <input class="username" name="username" type="text" placeholder="输入姓名查找" v-model="username">
                    <input class="search" type="button" value="查找" v-on:click="onSearch(username)">
                </form>
                <button class="add" href="javascript:;" v-on:click="addItem">新增</button>
            </div>
            <table>
                <tr>
                    <th><input type="checkbox" v-model="isCheckedAll" v-on:click="checkAll">&nbsp;<a href="javascript:;" @click="orderTableData('id')">id</a></th>
                    <th><a href="javascript:;" @click="orderTableData('name')">姓名</a></th>
                    <th><a href="javascript:;" @click="orderTableData('age')">年龄</a></th>
                    <th><a href="javascript:;" @click="orderTableData('tel')">手机号</a></th>
                    <th>操作</th>
                </tr>
                <tr v-for="(item, index) in filterTableData">
                    <td><input type="checkbox" v-bind:value="item.id" v-model="checkedList">&nbsp;<label>{{ item.id }}</label></td>
                    <td>{{ item.name }}</td>
                    <td>{{ item.age }}</td>
                    <td>{{ item.tel }}</td>
                    <td>
                        <a href="javascript:;" v-on:click="editItem(index)">编辑</a>
                        <span>&nbsp;&nbsp;</span>
                        <a href="javascript:;" v-on:click="deleteItem(index)">删除</a>
                    </td>
                </tr>
            </table>
        </div>
        <div class="edit" v-show="isShow === 'edit' || isShow === 'add'">
            <form action="" name="editForm">
                <div class="edit-input username"><label for="username">姓名</label><input type="text" name="username" v-model="editItemNow.name"></div>
                <div class="edit-input age"><label for="age">年龄</label><input type="text" name="age" v-model="editItemNow.age"></div>
                <div class="edit-input tel"><label for="tel">电话</label><input type="text" name="tel" v-model="editItemNow.tel"></div>
                <div class="edit-input edit-input-btn">
                    <input class="submit" type="button" value="确定" @click="submitItem">
                </div>
            </form>
        </div>
    </div>
</template>

<script>
    import axios from 'axios'
    export default {
        data() {
            return {
                tableData: [],  //数据列表
                username: '',   //用户名
                query: '',      //检索关键字
                editItemNow: {},    //当前item
                isShow: 'list', //显示列表页还是编辑页
                order: 'asc',   //排序方式
                id: '',         //列表最后一项数据id值，用于自增id
                checkedList: [],    //选中列表
                isCheckedAll: ''  //是否全选
            }
        },
        methods: {
            onSearch(searchQuery) {
                this.query = searchQuery;
            },
            deleteItem(index) {
                this.tableData.splice(index, 1);
            },
            editItem(index) {
                this.isShow = 'edit';
                this.editItemNow = this.filterTableData[index];
                this.editItemNow.index = index;
            },
            addItem() {
                this.isShow = 'add';
                this.editItemNow = {};
            },
            submitItem() {
                this.isShow = 'list';
                if ( this.editItemNow.index === undefined ) {
                    this.editItemNow.id = ++this.id;
                    this.filterTableData.push(this.editItemNow);
                } else {
                    this.filterTableData[this.editItemNow.index] = this.editItemNow;
                }
            },
            orderTableData(key) {
                this.order = this.order === 'asc' ? 'desc' : 'asc';
                this.tableData = this._.orderBy(this.tableData, [key], [this.order])
            },
            checkAll: function() {
                var _this = this;
                if (this.isCheckedAll) {//实现反选
                    _this.checkedList = [];
                    _this.tableData.forEach(function(item) {
                        _this.checkedList.push(item.id);
                    });
                }else{//实现全选
                    _this.checkedList = [];
                }
            }
        },
        watch: {
            'checkedList': {
                handler: function (val, oldVal) {
                    if (this.checkedList.length === this.tableData.length) {
                        this.isCheckedAll=true;
                    }else{
                        this.isCheckedAll=false;
                    }
                },
                deep: true
            }
        },
        computed: {
            filterTableData() {
                const self = this;
                return self.tableData.filter(function (item) {
                    return (item.name.indexOf(self.query) > -1)
                })
            }
        },
        beforeMount(){
            axios({
                method: 'get',
                url: '/static/tabledemo.json',
                responseType: 'json',
            }).then( (res) => {
                this.tableData = res.data.data;
                this.id = this.tableData[this.tableData.length - 1].id;
            } ).catch( (error) => {
                console.log(error);
            } );
        }
    }
</script>
<style scoped>
    * { margin: 0; padding: 0; font-size: 16px;}
    input[type="text"] { width: 200px; height: 20px; padding: 5px; vertical-align: middle;}
    input[type="button"] { width: 80px; height: 36px;}
    table { width: 100%; border-collapse: collapse; text-align: center;}
    table, tr, td, th { border: 1px solid #ccc;}
    td, th { height: 40px; line-height: 40px;}
    tr:hover { background-color: #f6facc; transition: 0.5s ease;}
    td a:hover { text-decoration: underline;}
    .filter { margin-bottom: 20px; position: relative;}
    .filter .username { width: 300px; height: 20px; padding: 5px; vertical-align: middle;}
    .filter .search { width: 80px; height: 34px; vertical-align: middle;}
    .filter .add { position: absolute; top: 0; right: 0; width: 80px; height: 34px; line-height: 34px; text-align: center; cursor: pointer;}
    .edit { padding: 20px;}
    .edit .edit-input { margin-bottom: 10px;}
    .edit .edit-input label { width: 150px; margin-right: 20px;}
    .edit-input-btn .submit{ margin-left: 100px;}
</style>
