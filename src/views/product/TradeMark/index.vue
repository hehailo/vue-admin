<template>
  <div>
    <el-button
      type="primary"
      icon="el-icon-plus"
      style="margin: 10px 0px"
      @click="showDialog"
      >增加</el-button
    >
    <!-- 表格 -->
    <el-table :data="list" style="width: 100%" border>
      <el-table-column
        type="index"
        width="180px"
        prop="序号"
        label="序号"
        align="center"
      >
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称" width="width">
      </el-table-column>
      <el-table-column prop="logoUrl" label="品牌logo" width="width">
        <template slot-scope="{ row }">
          <img :src="row.logoUrl" style="width: 100px; height: 100px" />
        </template>
      </el-table-column>
      <el-table-column prop="prop" label="操作" width="width">
        <template slot-scope="{ $index, row }">
          <el-button
            size="mini"
            @click="handleEdit($index, row)"
            icon="el-icon-edit"
            type="warning"
            >修改</el-button
          >
          <el-button
            size="mini"
            type="danger"
            icon="el-icon-delete"
            @click="handleDelete($index, row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      style="margin-top: 20px; text-align: center"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="page"
      :page-sizes="[2, 3, 5, 10]"
      :page-size="limit"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    >
    </el-pagination>

    <!-- 弹框 -->
    <el-dialog
      :title="this.trademarkForm.id ? '修改品牌' : '添加品牌'"
      :visible.sync="dialogFormVisible"
    >
      <el-form :model="trademarkForm">
        <el-form-item label="品牌名称" :label-width="formLabelWidth">
          <el-input
            v-model="trademarkForm.tmName"
            autocomplete="off"
            style="width: 80%"
          ></el-input>
        </el-form-item>
        <!-- 图片上传 -->
        <el-form-item label="品牌LOGO" :label-width="formLabelWidth">
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img
              v-if="trademarkForm.logoUrl"
              :src="trademarkForm.logoUrl"
              class="avatar"
            />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div slot="tip" class="el-upload__tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <!-- 弹框部分 -->
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "TradeMark",
  data() {
    return {
      page: 1,
      limit: 3,
      total: 0,
      list: [],
      total: 0,
      dialogFormVisible: false,
      trademarkForm:'',
      trademarkForm: { tmName: "", logoUrl: "" },
      formLabelWidth: "120px",
    };
  },
  mounted() {
    this.getData();
  },
  methods: {
    async getData() {
      let { page, limit } = this;
      let result = await this.$API.tradeMark.reqTradeMarkList(page, limit);
      if (result.code == 200) {
        //分别是展示数据总条数与列表展示的数据
        this.total = result.data.total;
        this.list = result.data.records;
      }
    },
    showDialog() {
      this.dialogFormVisible = true;
      //清除数据  解决回显问题
      this.trademarkForm= { tmName: "", logoUrl: "" };
    },

    // 操作按钮
    // 修改
    handleEdit($index, row) {
      this.dialogFormVisible = true;
      console.log("row",row);
      // 会导致页面产生同步的更改
      // 使用浅拷贝
      // this.trademarkForm= row;
      this.trademarkForm= {...row}
    },
    // 删除
    handleDelete($index, row) {
      this.$confirm(`你确定删除${row.tmName}, 是否继续?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(async () => {
          let result = await this.$API.tradeMark.reqDeleteTradeMark(row.id);
          if(result.code == 200){
            // 当前页面的数据大于1  则返回上一页
            if(this.list.length <= 1 &&  this.page != 1){
                this.page = this.page-1
            }
            this.getData();
            // 点击确定
            this.$message({
              type: 'success',
              message: '删除成功!'
            });
          }else{
            // 点击确定
            this.$message({
              type: 'error',
              message: '删除失败! 请重新尝试'
            });
          }
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        });
    },
    // 分页器有关
    handleSizeChange(limit) {
      this.limit = limit;
      this.page = 1;
      this.getData();
    },
    handleCurrentChange(page) {
      this.page = page;
      this.getData();
    },
    // 处理图片有关
    handleAvatarSuccess(res, file) {
      console.log("l23233");
      this.trademarkForm.logoUrl = res.data;
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },

    //添加按钮（添加品牌|修改品牌）
    async addOrUpdateTradeMark() {
      this.dialogFormVisible = false;
      //发请求（添加品牌|修改品牌）
      console.log("trademarkForm",this.trademarkForm);
      let result = await this.$API.tradeMark.reqAddOrUpdateTradeMark(
        this.trademarkForm
      );
      if (result.code == 200) {
        //弹出信息:添加品牌成功、修改品牌成功
        this.$message({
          type: "success",
          message: this.trademarkForm.id ? "修改品牌成功" : "添加品牌成功",
        });
        //添加或者修改品牌成功以后，需要再次获取品牌列表进行展示
        //如果添加品牌： 停留在第一页，修改品牌应该留在当前页面
        if(!this.trademarkForm.id){ //添加
          this.page = 1;
        }
        this.getData(this.trademarkForm.id ? this.page : 1);
      }
    },
  }
};
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>