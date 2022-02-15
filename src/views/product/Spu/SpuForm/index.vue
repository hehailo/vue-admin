<template>
  <div>
    <!-- model 收集到的数据 -->
    <el-form ref="form" :model="spu" label-width="80px">
      <el-form-item label="SPU名称" class="inputWidth">
        <el-input placeholder="SPU名称" v-model="spu.spuName"></el-input>
      </el-form-item>
      <!-- 品牌下拉框 -->
      <el-form-item label="品牌" class="inputWidth">
        <el-select placeholder="请选择品牌" v-model="spu.tmId">
          <el-option
            :label="tm.tmName"
            :value="tm.id"
            v-for="tm in tradeMarkList"
            :key="tm.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="SPU描述" class="inputWidth">
        <el-input
          type="textarea"
          rows="4"
          placeholder="描述"
          v-model="spu.description"
        ></el-input>
      </el-form-item>
      <!-- 照片墙 -->
      <el-form-item label="SPU图片" style="width:90%">
        <!-- upload组件 -->
        <el-upload
          action="/dev-api/admin/product/fileUpload"
          list-type="picture-card"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :file-list="spuImageList"
          :on-success="handleAvatarSuccess"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <!-- 预览 放大框 -->
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="" />
        </el-dialog>
      </el-form-item>
      <!-- 销售属性 -->
      <el-form-item label="销售属性">
        <!-- 销售属性下拉列表 可选择 -->
        <el-select
          v-model="attrId"
          :placeholder="`还有${canSelectAttr.length}未选择`"
        >
          <el-option
            :label="canselect.name"
            :value="`${canselect.name}:${canselect.id}`"
            v-for="canselect in canSelectAttr"
            :key="canselect.id"
          >
          </el-option>
        </el-select>
        <!-- attrId 是下拉框手机的数据 没有数据就是没有选择 需要给他置灰 -->
        <el-button style="margin-left: 10px" type="primary" :disabled="!attrId"
          >+ 增加销售属性</el-button
        >
        <!-- 增加销售属性 下拉列表 -->
        <el-table
          :data="spu.spuSaleAttrList"
          style="width: 80%; margin-top: 5px"
          border
        >
          <el-table-column type="index" label="序号" width="100">
          </el-table-column>
          <el-table-column prop="saleAttrName" label="属性名" width="200">
          </el-table-column>
          <!-- 属性名称列表  -->
          <el-table-column prop="prop" label="属性名称列表" width="width">
            <template slot-scope="{ row }">
              <el-tag
                :key="tag.id"
                v-for="(tag,index) in row.spuSaleAttrValueList"
                closable
                :disable-transitions="true"
                type="warning"
                effect="dark"
                @close="handleClose(row.spuSaleAttrValueList,index)"
              >
                {{ tag.saleAttrValueName }}
              </el-tag>
              <el-input
                class="input-new-tag"
                v-if="inputVisible"
                v-model="inputValue"
                ref="saveTagInput"
                size="small"
                @keyup.enter.native="handleInputConfirm"
                @blur="handleInputConfirm"
              >
              </el-input>
              <el-button
                v-else
                class="button-new-tag"
                size="small"
                @click="showInput"
                >+ 新增</el-button
              >
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="200">
            <template slot-scope="{ row }">
              <el-button type="danger" icon="el-icon-delete"></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item label="">
        <el-button type="primary" size="medium">保存</el-button>
        <el-button size="medium" @click="$emit('changeScene', 0)">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "SpuForm",
  data() {
    return {
      dialogImageUrl: "",
      dialogVisible: false,
      msg: "",
      tradeMarkList: [],
      spu: {
        description: "",
        spuName: "",
        spuSaleAttrList: [],
      },
      spuImageList: [],
      title: "",
      spuName: "",
      tmId: "",
      //全部属性
      saleAttrList: [],
      attrId: "",
      inputVisible: false,
      inputValue: "",
    };
  },
  methods: {
    async initData(spu) {
      console.log("spu", spu);
      //   获取品牌列表
      //获取SPU信息的数据
      let spuResult = await this.$API.spu.reqSpu(spu.id);
      if (spuResult.code == 200) {
        //在修改spu的时候,需要向服务器发请求的，把服务器返回的数据（对象），赋值给spu属性
        console.log("spu信息", spuResult.data);
        this.spu = spuResult.data;
      }
      //获取品牌的信息
      let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
      if (tradeMarkResult.code == 200) {
        // console.log("品牌的信息", tradeMarkResult.data);
        this.tradeMarkList = tradeMarkResult.data;
      }
      //获取spu图片的数据
      let spuImageResult = await this.$API.spu.reqSpuImageList(spu.id);
      if (spuImageResult.code == 200) {
        let listArr = spuImageResult.data;
        // console.log("照片墙数据", spuImageResult.data);
        //由于照片墙显示图片的数据需要数组，数组里面的元素需要有name与url字段
        //需要把服务器返回的数据进行修改
        listArr.forEach((item) => {
          item.name = item.imgName;
          item.url = item.imgUrl;
        });
        //把整理好的数据赋值给
        console.log("listArr", listArr);
        this.spuImageList = listArr;
      }
      //获取平台全部的销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        console.log("全部的销售属性", saleResult.data);
        this.saleAttrList = saleResult.data;
      }
    },

    // 处理照片墙有关
    // 删除图片
    handleRemove(file, fileList) {
      this.spuImageList = fileList;
    },
    // 预览图片
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    // 成功上传图片
    handleAvatarSuccess(response, file, fileList) {
      fileList[fileList.length - 1].url = response.data;
      this.spuImageList = fileList;
      console.log("成功上传图片的fileList", fileList);
    },

    // 动态编辑标签
    handleClose(tagList,index) {
      tagList.splice(index,1);
    },

    showInput() {
      this.inputVisible = true;
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },

    handleInputConfirm() {
      let inputValue = this.inputValue;
      if (inputValue) {
        this.dynamicTags.push(inputValue);
      }
      this.inputVisible = false;
      this.inputValue = "";
    },
  },
  computed: {
    //可选属性
    canSelectAttr() {
      return this.saleAttrList.filter((item) => {
        return this.spu.spuSaleAttrList.every((item2) => {
          return item.name != item2.saleAttrName;
        });
      });
    },
  },
};
</script>

<style scoped>
.inputWidth {
  width: 600px;
}
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>
