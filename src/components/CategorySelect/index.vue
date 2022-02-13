<template>
  <div>
    <el-form :inline="true" :model="cForm" class="demo-form-inline">
      <el-form-item label="一级分类">
        <el-select
          v-model="cForm.category1Id"
          placeholder="请选择"
          @change="getList2"
          :disabled="!show"
        >
          <el-option
            :label="item.name"
            :value="item.id"
            v-for="item in list1"
            :key="item.id"
          ></el-option>
          <el-option label="区域二" value="beijing"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级分类">
        <el-select
          v-model="cForm.category2Id"
          placeholder="请选择"
          @change="getList3"
            :disabled="!show"
        >
          <el-option
            :label="item2.name"
            :value="item2.id"
            v-for="item2 in list2"
            :key="item2.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="三级分类">
        <el-select v-model="cForm.category3Id" placeholder="请选择" :disabled="!show">
          <el-option
            :label="item3.name"
            :value="item3.id"
            v-for="item3 in list3"
            :key="item3.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          @click="onSubmit"
          :disabled="!cForm.category3Id && cForm.category3Id != '0'"
          >查询</el-button
        >
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { mapState } from "vuex";
export default {
  name: "CategorySelect",
  data() {
    return {
      //一级分类的数据
      list1: [],
      //二级分类的数据
      list2: [],
      //三级分类的数据
      list3: [],
      //表单收集的数据
      //收集相应的一级二级三级分类的id
      cForm: {
        category1Id: "",
        category2Id: "",
        category3Id: "",
      },
    };
  },
  computed: {
    ...mapState([""]),
  },
  props:["show"],
  mounted() {
    this.getData();
  },
  methods: {
    async getData() {
      //获取一级分类的请求：不需要携带参数
      let result = await this.$API.attr.reqCategory1List();
      console.log("list1", result);
      if (result.code == 200) {
        this.list1 = result.data;
      }
    },
    async getList2() {
      this.list2 = [];
      this.list3 = [];
      this.cForm.category2Id = "";
      this.cForm.category3Id = "";
      const { category1Id } = this.cForm;
      //   this.$emit("getCategoryId", { categoryId: category1Id, level: 1 });
      //通过一级分类的id，获取二级分类的数据
      let result = await this.$API.attr.reqCategory2List(category1Id);
      console.log("list2", result);
      if (result.code == 200) {
        this.list2 = result.data;
      }
    },
    async getList3() {
      this.list3 = [];
      this.cForm.category3Id = "";
      const { category2Id } = this.cForm;
      // this.$emit("getCategoryId", { categoryId: category2Id, level: 2 });
      let result = await this.$API.attr.reqCategory3List(category2Id);
      console.log("list3", result);
      if (result.code == 200) {
        this.list3 = result.data;
      }
    },
    onSubmit() {
      this.$emit("getCategoryId", { ...this.cForm });
    },
  },
};
</script>

<style scoped>
    .el-form-item{
        margin-bottom: 10px;
    }
</style>
