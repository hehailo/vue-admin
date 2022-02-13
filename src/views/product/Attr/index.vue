<template>
  <div>
    <!-- 上部分 -->
    <el-card style="margin: 10px 10px">
      <CategorySelect @getCategoryId="getListData" :show="!isShowTable"></CategorySelect>
    </el-card>
    <!-- 下部分 -->
    <el-card style="margin: 20px 10px">
      <!-- 展示列表 -->
      <div v-show="isShowTable">
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="addAttr"
          style="margin: 10px 0px"
          size="medium"
          >添加属性</el-button
        >
        <el-table :data="attrList" border style="width: 100%">
          <el-table-column type="index" label="序号" width="120">
          </el-table-column>
          <el-table-column prop="attrName" label="属性名称" width="250">
          </el-table-column>
          <el-table-column prop="name" label="属性值列表" width="width">
            <template slot-scope="{ row }">
              <el-tag
                v-for="attrValue in row.attrValueList"
                :key="attrValue.id"
                type="success"
                style="margin: 0 20px"
              >
                {{ attrValue.valueName }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="address" label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <el-button
                size="mini"
                icon="el-icon-edit"
                @click="editAttrValue(row)"
                >编辑</el-button
              >
              <el-button size="mini" type="danger" icon="el-icon-delete"
                >删除</el-button
              >
            </template>
          </el-table-column>
        </el-table>
      </div>
      <!-- 添加 修改区域 -->
      <div v-show="!isShowTable">
        <el-form
          :inline="true"
          class="demo-form-inline"
          style="margin-bottom: 10px"
        >
          <el-form-item label="属性名">
            <el-input
              v-model="attrInfo.attrName"
              placeholder="请输入属性名"
            ></el-input>
          </el-form-item>
        </el-form>
        <!-- 添加属性按钮 -->
        <el-button size="medium" type="primary" @click="addAttrValue"
          >+ 添加属性值</el-button
        >
        <!-- 属性值 新增、修改列表  -->
        <el-table
          :data="attrInfo.attrValueList"
          border
          style="width: 100%; margin: 10px 0px"
        >
          <el-table-column type="index" label="序号" width="200">
          </el-table-column>
          <el-table-column prop="prop" label="属性值名称" width="width">
            <template slot-scope="{ row, $index }">
              <!-- 输入框 -->
              <el-input
                v-if="row.editFlag"
                @blur="toLook(row)"
                :ref="$index"
                v-model="row.valueName"
                placeholder="请输入"
                @keyup.native.enter="toLook(row)"
              ></el-input>
              <!-- 展示span -->
              <!-- span标签不能独占一行 点击空白无法触发事件 -->
              <span
                v-else
                @click="toEdit(row, $index)"
                style="display: block"
                >{{ row.valueName }}</span
              >
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" style="width: 30%">
            <template slot-scope="{ row, $index }">
              <template>
                <el-popconfirm
                  :title="`确定删除 ${row.valueName} 吗？`"
                  @onConfirm="deleteAttrValue($index)"
                >
                  <!-- reference	触发 Popconfirm 显示的 HTML 元素 -->
                  <el-button
                    size="mini"
                    icon="el-icon-delete"
                    type="danger"
                    slot="reference"
                    >删除</el-button
                  >
                </el-popconfirm>
              </template></template
            ></el-table-column
          >
        </el-table>
        <!-- 保存、取消按钮 -->
        <el-button type="primary" @click="addOrUpadateAttr" :disabled="attrInfo.attrValueList.length<1">保存</el-button>
        <el-button @click="isShowTable = true">取消 </el-button>
      </div>
    </el-card>
  </div>
</template>
</template>

<script>
import cloneDeep from "lodash/cloneDeep";
export default {
  name: "Attr",
  data() {
    return {
      addFlag: false,
      isShowTable: true,
      category1Id: "",
      category2Id: "",
      category3Id: "",
      attrList: [],
      attrInfo: {
        attrName: "", //属性名
        attrValueList: [
          //属性值，因为属性值可以有多个因此用数组，每一个属性值都是一个对象需要attrId，valueName
        ],
        categoryId: 0, //三级分类的id
        categoryLevel: 3, //因为服务器也需要区分几级id
      },
    };
  },
  methods: {
    getListData(params) {
      this.addFlag = true;
      this.isShowTable = true;
      let { category1Id, category2Id, category3Id } = params;
      this.category1Id = category1Id;
      this.category2Id = category2Id;
      this.category3Id = category3Id;
      this.getAttrList();
    },
    // 查询列表数据
    async getAttrList() {
      let { category1Id, category2Id, category3Id } = this;
      let result = await this.$API.attr.reqAttrList(
        category1Id,
        category2Id,
        category3Id
      );
      if (result.code == 200) {
        console.log("attr列表表格", result.data);
        this.attrList = result.data;
      }
    },
    //添加属性
    addAttr() {
      this.isShowTable = false;
      // 把选中的三级id传过来
      this.attrInfo = {
        attrName: "", //属性名
        attrValueList: [
          //属性值，因为属性值可以有多个因此用数组，每一个属性值都是一个对象需要attrId，valueName
        ],
        categoryId: this.category3Id, //三级分类的id
        categoryLevel: 3, //因为服务器也需要区分几级id
      };
    },
    // 添加属性值
    addAttrValue() {
      this.attrInfo.attrValueList.push({
        editFlag: true,
      });
      this.$nextTick(() => {
        this.$refs[this.attrInfo.attrValueList.length - 1].focus();
      });
    },
    // 修改属性值
    editAttrValue(row) {
      this.isShowTable = false;
      console.log(row);
      // this.attrInfo = {...row};
      // 上面使用浅拷贝 对象里面的数组的元素的值改变 会影响原理的页面
      // 这个值还是指向一样的地址 所以使用深拷贝
      this.attrInfo = cloneDeep(row);
      this.attrInfo.attrValueList.forEach((item) => {
        this.$set(item, "editFlag", false);
      });
    },
    //转变为读
    toLook(row) {
      row.editFlag = false;
    },
    // 转变为写
    toEdit(row, $index) {
      console.log("toEDit", $index);
      row.editFlag = true;
      this.$nextTick(() => {
        console.log($index);
        this.$refs[$index].focus();
      });
    },
    //添加、修改、删除后保存
    async addOrUpadateAttr() {
      // 去除空白属性
      // 去除 editFlag

      this.attrInfo.attrValueList = this.attrInfo.attrValueList.filter((item) => {
        if (!item.valueName == "") {
          delete item.editFlag;
          return true;
        }
      });
      console.log("params", this.attrInfo);
      try {
        //发请求
        await this.$API.attr.reqAddOrUpdateAttr(this.attrInfo);
        // 提示成功
        this.$message({ type: "success", message: "保存成功" });
        // 展示表格
        this.isShowTable = true;
        // 刷新数据
        this.getAttrList();
      } catch (error) {
        this.$message({ type: "error", message: "保存失败！" });
      }
    },
    // 删除、添加、修改
    deleteAttrValue($index) {
      this.attrInfo.attrValueList.splice($index, 1);
    },
  },
};
</script>

<style scoped>
.el-form-item {
  margin-bottom: 0px;
}
</style>