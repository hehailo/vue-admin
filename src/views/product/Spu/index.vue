<template>
  <div>
    <el-card style="margin: 10px 0px;padding:10px">
      <CategorySelect @getCategoryId="getListData"></CategorySelect>
    </el-card>
    <el-card style="margin: 10px 0px;padding:10px">
      <div v-show="scene == 0">
        <el-button
          type="primary"
          style="margin-top: 10px"
          :disabled="!category3Id"
          @click="addSpu"
          >+ 添加SPU</el-button
        >
        <el-table
          :data="spulist"
          style="width: 100%; margin-top: 10px"
          border=""
        >
          <el-table-column
            type="index"
            label="序号"
            width="200px"
            align="center"
          ></el-table-column>
          <el-table-column prop="spuName" label="SPU名称" width="width">
          </el-table-column>
          <el-table-column prop="description" label="SPU描述" width="width">
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{ row }">
              <!-- 这里按钮将来用hintButton替换 -->
              <hint-button
                type="success"
                icon="el-icon-plus"
                size="mini"
                title="添加sku"
                @click="addSku(row)"
              ></hint-button>
              <hint-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                title="修改spu"
                @click="updateSpu(row)"
              ></hint-button>
              <hint-button
                type="info"
                icon="el-icon-info"
                size="mini"
                title="查看当前spu全部sku列表"
                @click="handler(row)"
              ></hint-button>
              <el-popconfirm
                title="这是一段内容确定删除吗？"
                @onConfirm="deleteSpu(row)"
              >
                <hint-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  title="删除spu"
                  slot="reference"
                ></hint-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <!-- 分页器 -->
        <el-pagination
          style="text-align: center; margin-top: 10px"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="page"
          :page-sizes="[10, 20, 50]"
          :page-size="limit"
          layout=" prev, pager, next, jumper,->,sizes,total"
          :total="total"
        >
        </el-pagination>
      </div>
      <!-- 点击 修改、添加的时候 切换场景 -->
      <SpuForm
        v-show="scene == 1"
        ref="spu"
        @changeScene="changeScene"
      ></SpuForm>
      <SkuForm v-show="scene == 2" ref="sku"></SkuForm>
    </el-card>
  </div>
</template>

<script>
import SkuForm from "./SkuForm";
import SpuForm from "./SpuForm";
export default {
  name: "Spu",
  data() {
    return {
      scene: 0,
      category1Id: "",
      category2Id: "",
      category3Id: "",
      page: 1,
      limit: 3,
      total: 0,
      spulist: [],
    };
  },
  components: {
    SkuForm,
    SpuForm,
  },
  methods: {
    getListData(params) {
      let { category1Id, category2Id, category3Id } = params;
      this.category1Id = category1Id;
      this.category2Id = category2Id;
      this.category3Id = category3Id;
      this.getSpuList();
    },
    async getSpuList() {
      const { page, limit, category3Id } = this;
      //携带三个参数:page 第几页  limit 每一页需要展示多少条数据  三级分类id
      let result = await this.$API.spu.reqSpuList(page, limit, category3Id);
      if (result.code == 200) {
        this.total = result.data.total;
        this.spulist = result.data.records;
        console.log("SPU列表", result);
      }
    },
    handleCurrentChange(page) {
      console.log("page", page);
    },
    handleSizeChange(limit) {
      console.log("limit", limit);
    },

    addSku(row) {
      this.scene = 2;
    },
    addSpu() {
      this.scene = 1;
    },
    updateSpu(row) {
      this.scene = 1;
      //给组件打一个ref 就可以获取组件实例对象
      // 可以操作子组件的方法和数据
      this.$refs.spu.initData(row);
    },
    // 自定义事件回调
    // 子组件取消
    changeScene(scene) {
      this.scene = scene;
    },
    handler(row) {},
    deleteSpu(row) {},
  },
};
</script>

<style>
</style>