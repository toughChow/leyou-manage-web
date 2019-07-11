<template>
  <div>
    <v-card>
      <!-- 卡片的头部 -->
      <v-card-title>
        <v-btn @click="addBrand" color="primary">新增</v-btn>
        <!--空间隔离组件-->
        <v-spacer />
        <!--搜索框，与search属性关联-->
        <v-text-field label="输入关键字搜索" v-model="search" append-icon="search" hide-details/>
      </v-card-title>
      <!-- 分割线 -->
      <v-divider/>
      <!--卡片的中部-->
      <v-data-table
        :headers="headers"
        :items="brands"
        :pagination.sync="pagination"
        :total-items="totalBrands"
        :loading="loading"
        class="elevation-10"
        select-all
      >
<!--        v-model="selected"-->
        <template slot="items" slot-scope="props">
          <td>{{ props.item.id }}</td>
          <td class="text-xs-center">{{ props.item.name }}</td>
          <td class="text-xs-center"><img :src="props.item.image"></td>
          <td class="text-xs-center">{{ props.item.letter }}</td>
          <td class="justify-center layout">
            <v-btn color="info" @click="editBrand(props.item)">编辑</v-btn>
            <v-btn color="warning">删除</v-btn>
          </td>
        </template>
      </v-data-table>
    </v-card>

    <v-dialog
      v-model="dialog"
      width="500"
    >
      <v-card>
        <v-toolbar dark dense color="primary">
          <v-toolbar-title>{{isEdit ? '修改品牌' : '新增品牌'}}</v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="dialog = false">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text class="px-5 py-2">
          <!-- 表单 -->
          <brand-form :oldBrand="oldBrand" :isEdit="isEdit" @close="dialog = false" :reload="reload"/>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
  import BrandForm from './BrandForm'
  export default {
    name:"MyBrand",
      /*props:{
        oldBrand:{
          type:Object
        }
      },*/
    components: {
      BrandForm
    },
    data () {
      return {
        headers: [
          { text: "品牌id", value: "id", align: "center", sortable: true},
          { text: "品牌名称", value: "name", align: "center", sortable: false},
          { text: "品牌LOGO", value: "image", align: "center", sortable: false},
          { text: "品牌首字母", value: "letter", align: "center", sortable: true},
          { text: "操作", align: "center", sortable: false},
        ],
        brands: [

        ],
        brand:{},
        oldBrand:{},
        pagination: {},
        totalBrands: 0,
        loading: false,
        search:'',
        dialog: false,
        isEdit: false,
      }
    },
    created(){
      this.getDataFromServer();
    },
    methods: {
      addBrand() {
        this.brand = {};
        this.dialog = true;
        this.isEdit = false;
      },
      getDataFromApi() {
        this.loading = true;
        // 200ms后返回假数据
        window.setTimeout(() => {
          this.items = brandData.slice(0,4);
          this.totalItems = 100;
          this.loading = false;
        }, 200)
      },
      getDataFromServer() {
        this.loading = true;

        //发起ajax请求
        // 分页查询page,rows,key,sortBy,desc

        this.$http.get("/item/brand/page",{
          params:{
            page:this.pagination.page,
            rows:this.pagination.rowsPerPage,
            sortBy:this.pagination.sortBy,
            desc:this.pagination.descending,
            key:this.search,
          }
        }).then(resp =>{
          this.brands=resp.data.items;
          this.totalBrands = resp.data.total;
          //关闭进度条
          this.loading = false;
        })
      },
      editBrand(oldBrand) {
        console.log(oldBrand)
        this.oldBrand = oldBrand
        this.isEdit = true;
        this.dialog = true;
      },
      reload(){
        //关闭对话框
        this.show=false;
        //刷新页面
        this.getDataFromServer();
      },
    }
  }
</script>

<style scoped>

</style>
