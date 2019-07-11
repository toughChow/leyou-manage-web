<template>
  <v-form v-model="valid" ref="brandForm">
    <v-text-field
      label="品牌名称"
      v-model="brand.name"
      :rules="[v => !!v || '品牌名称不能为空']"
      :counter="10"
      required
    />
    <v-text-field
      label="首字母"
      v-model="brand.letter"
      :rules="[v => v.length === 1 || '首字母只能是1位']"
      required
      mask="A"
    />
    <v-cascader url="/item/category/list" required
                v-model="brand.categories"
                multiple label="商品分类"/>
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 16px; color: #444">品牌LOGO：</span>
      </v-flex>
      <v-flex>
        <v-upload
          v-model="brand.image" ref="image_url" url="/upload/image" :multiple="false" :pic-width="250" :pic-height="90"
        />
      </v-flex>
    </v-layout>
    <v-layout class="my-4">
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear" color="warning">重置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
  import config from '@/config';
  export default {
    // name: "brand-form",
    props: {
      oldBrand: Object,
      isEdit: {
        type: Boolean,
        default: false
      },
      dialog: {
        type: Boolean,
        default: true
      }
    },
    data() {
      return {
        baseUrl: config.api,
        valid:false,
        brand: {
          name: '',
          image: '',
          letter: '',
          categories: []
        },
        name:'',
        imageDialogVisible:false
      }
    },
    watch: {
      pagination: {
        deep:true,
        handler() {
          this.getDataFromServer();
        }
      },
      oldBrand: {
        handler(val) {
          if (val) {
            console.log(val)
            console.log(666)
            // this.brand = Object.deepCopy(val);
            this.name = val.name;
            this.brand.image = val.image;
            this.brand.letter = val.letter;
            this.brand.categories = val.categories;
          } else {
            this.brand = {
              name:'',
              letter:'',
              image:'',
              categories:[],
            }
          }
        },
        deep:true
      }
    },
    methods: {
      submit() {
        // 表单校验
        if (this.$refs.brandForm.validate()) {

          /**
           * 使用解构表达式获取数据，除categories以外的数据都放入rest中，然后对categories使用map进行处理，得到id后重新赋值给
           * rest里面的categories数组
           */
          const {categories, ... rest}=this.brand;
          rest.categories=categories.map(c => c.id).join(",");
          console.log(rest)
          // 将数据提交到后台
          this.$http({
            method: this.isEdit ? 'put' : 'post',
            url: '/item/brand',
            data: this.$qs.stringify(rest),
          }).then(() => {
            // 关闭窗口
            this.$message.success("保存成功！");
            this.closeWindow();
            this.emit('reload')
          }).catch((e) => {
            console.log(e)
            this.$message.error("保存失败！");
          });
        }
      },
      clear() {
        // 重置表单
        this.$refs.brandForm.reset();
        this.categories = [];
      },
      // 图片上传出成功后操作
      handleImageSuccess(res) {
        this.brand.image = res;
      },
      removeImage(){
        this.brand.image = "";
      },
      closeWindow(){
        this.$emit("close");
        //重置表单
        this.brand.name='';
        this.brand.letter='';
        this.brand.image='';
        this.brand.categories=[];
        console.log(this.$refs)
        this.$refs.image_url._data.dialogImageUrl='';
      }
    }
  }
</script>

<style scoped>

</style>
