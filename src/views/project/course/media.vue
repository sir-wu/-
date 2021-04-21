<template>
<div>
    <div>
       
  <el-row class="main" :gutter="20">
      <el-col :span="6">
  <el-button class="button" type="primary" @click="dialogFormVisible = true"> <i class="el-icon-edit"></i>新增图文</el-button>
</el-col>
 <el-col :span="12">
  <el-select v-model="value" placeholder="商品状态" class="select">
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value">
    </el-option>
  </el-select>
  </el-col >
    <el-col :span="6" class="wocao">
  <el-input v-model="listQuery.title" placeholder="标题" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
 <el-button  class="searchB" type="primary" @click="handleFilter"> <i class="el-icon-search"></i>搜索</el-button>
 </el-col>
</el-row>

 <el-table :data="tableData" style="width: 100%" border v-loading="listLoading">
      <el-table-column sortable prop="id" label="ID" width="100" align="center">
      </el-table-column>
      <el-table-column  label="图文内容" width="500">
        <template slot-scope="{ row }">
          <div style="display: flex">
            <img
              :src="row.cover"
              style="width: 100px; height: 50px; margin-right: 10px"
            />
            <div
              style="
                display: flex;
                flex-direction: column;
                justify-content: space-between;
              "
            >
              <span>{{ row.title }}</span>
              <span style="color: red">￥{{ row.price }}</span>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column
        prop="sub_count"
        label="订阅量"
        width="100"
        align="center"
      >
      </el-table-column>


      <el-table-column label="状态" class-name="status-col" width="100">
    <template slot-scope="{row}">
        <el-tag :type="row.status ? 'success' : 'danger'">
            {{ row.status | statusFilter }}
        </el-tag>
    </template>
     </el-table-column>


      <el-table-column prop="created_time" label="创建时间" align="center">
      </el-table-column>
      <el-table-column prop="try" label="操作" align="center">
           <template slot-scope="{row,$index}">
          <el-button type="primary" size="mini"  @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button v-if="row.status == 0" size="mini" type="success" @click="handleModifyStatus(row,1)">
            上架
          </el-button>
          <el-button v-else-if="row.status == 1" size="mini" @click="handleModifyStatus(row,0)">
            下架
          </el-button>
          <el-popconfirm title="是否要删除该记录？" @onConfirm="handleDelete(row,$index)" style="margin-left:10px;">
               <el-button v-if="row.status!='deleted'" size="mini" type="danger" slot="reference">删除</el-button>
           </el-popconfirm>
          
        </template>
      </el-table-column>
    </el-table>

    <!-- 分类列表的分页区域 -->
            <div class="pagination-container">
                <el-pagination
                        @size-change="handleSizeChange"
                        @current-change="handleCurrentChange"
                        :current-page="listQuery.pageNum"
                        :page-sizes="[10,20,30,50]"
                        :page-size="listQuery.pageSize"
                        layout="total, sizes, prev, pager, next, jumper"
                        :total="total">
                </el-pagination>
            </div>
        </div>


    <!-- 增加商品区域 -->

   
    <el-dialog title="新增"  :visible.sync="dialogFormVisible" fullscreen>
  <el-form :model="temp" :rules="rules" ref="dataForm">
    <el-form-item label="标题" prop="title" :label-width="formLabelWidth" style="width:500px">
      <el-input v-model="temp.title" autocomplete="off"></el-input>
    </el-form-item>

<!-- 图片上传 -->
<el-form-item label="封面"style="margin-left:76px" >
    <el-upload
  action="https://jsonplaceholder.typicode.com/posts/"
  list-type="picture-card"
  :on-preview="handlePictureCardPreview"
  :on-remove="handleRemove"
  style="margin-left:50px"
  v-model="temp.fengmian"
  >
  <i class="el-icon-plus"></i>
</el-upload>
<el-dialog :visible.sync="dialogVisible">
  <img width="100%" :src="dialogImageUrl" alt="">
</el-dialog>
</el-form-item>

<el-form-item label="课程介绍" type="text" prop="name" :label-width="formLabelWidth" style="width:500px">
      <el-input v-model="temp.kechengjieshao" autocomplete="off"></el-input>
    </el-form-item>

<!-- 富文本 -->
<el-form-item   prop="try" label="试看内容"style="margin-left:76px" >
<div class="components-container" style="width:50%">
    <div>
      <tinymce v-model="temp.try" :height="300" />
    </div>
    <div class="editor-content" v-html="temp.trysee" />
  </div>
</el-form-item>

<el-form-item  prop="content" label="课程内容"style="margin-left:76px" >
<div class="components-container" style="width:50%">
    <div>
      <tinymce v-model="temp.content" :height="300" />
    </div>
    <div class="editor-content" v-html="temp.trysee" />
  </div>
</el-form-item>


<!-- 计数器 -->
<el-form-item  label="课程价格"style="margin-left:76px" >
  <el-input-number v-model="temp.price" @change="handleChange" :min="1" :max="10" label="描述文字"></el-input-number>
</el-form-item>

<!-- 状态 -->
<el-form-item  label="状态"style="margin-left:76px" >
  <el-radio-group v-model="temp.status">
<el-radio :label="0" >下架</el-radio>
  <el-radio :label="1">上架</el-radio>
   </el-radio-group>
  </el-form-item>
  </el-form>

  <template slot="footer" class="dialog-footer"></template>
    <el-button @click="dialogFormVisible = false">取 消</el-button>
    <el-button type="primary"  @click="dialogStatus==='create'?createData():updateData()">提交</el-button>
</el-dialog>
       
    </div>
    </div>
</template>

<script>
import {
  fetchList,
  deleteMedia,
  createMedia,
  updateMedia,
} from "../../../api/media";
import Tinymce from "@/components/Tinymce";
let defaultListQuery = {
  page: 1, // 当前是第几页
  limit: 10, // 每页5条数据
  title: undefined,
};

const statusOptions = {
  0: "已下架",
  1: "已上架",
};
export default {
  components: { Tinymce },
  data() {
    return {
      dialogStatus: "",
      temp: {
        id: undefined,
        title: "",
        status: 1,
        price: 0,
        try: "",
        content: "",
        cover: "",
      },

      listQuery: Object.assign({}, defaultListQuery),
      options: [
        {
          value: "选项1",
          label: "已上架",
        },
        {
          value: "选项2",
          label: "已下架",
        },
      ],
      rules: {
        title: [
          {
            required: true,
            message: "标题不能为空",
            trigger: "blur",
          },
        ],
        try: [
          {
            required: true,
            message: "试看内容不能为空",
            trigger: "blur",
          },
        ],
        content: [
          {
            required: true,
            message: "课程内容不能为空",
            trigger: "blur",
          },
        ],
      },
      tableKey: 0,
      sortOptions: [
        {
          label: "ID Ascending",
          key: "+id",
        },
        {
          label: "ID Descending",
          key: "-id",
        },
      ],
      ruleForm: {
        name: "",
        region: "",
        date1: "",
        date2: "",
        delivery: false,
        type: [],
        resource: "",
        desc: "",
      },

      textMap: {
        update: "修改",
        create: "新增",
      },
      dialogImageUrl: "",
      dialogVisible: false,

      dialogTableVisible: false,
      dialogFormVisible: false,
      list: null,
      form: {
        name: "",
        region: "",
        date1: "",
        date2: "",
        delivery: false,
        type: [],
        resource: "",
        desc: "",
      },
      formLabelWidth: "120px",
      value: "",
      input: "",
      total: null, //总记录数据
      statusOptions, //控制商品
      listLoading: false,
      tableData: [],
      formInline: {
        user: "",
        region: "",
      },
      num: 1,
      radio: "1",
    };
  },
  //把状态上架下架过滤出来
  filters: {
    statusFilter(status) {
      return statusOptions[status];
    },
  },
  methods: {
    // onSubmit() {
    //   console.log("submit!");
    // },
    // 改变每页显示的条数
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.listQuery.page = 1;
      this.listQuery.limit = val;
      // 重新加载数据
      this.getlist();
    },
    // 切换页码
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.listQuery.page = val;
      // 重新加载数据
      this.getlist();
    },
    //获取图文列表
    async getlist() {
      // console.log(this.defaultListQuery)
      this.listLoading = true;
      
      let result = await fetchList(this.listQuery);
      console.log(this.listQuery);
      console.log(result);
      if (result.code === 20000) {
        this.tableData = result.data.items;
        this.total = result.data.total;
         this.list = result.data.items
        // console.log(this.total)
        this.listLoading = false;
      }
    },

    //控制产品上架下架
    handleModifyStatus(row, status) {
      this.$message({
        message: "操作Success",
        type: "success",
      });
      row.status = status;
    },
    //删除商品
    handleDelete(row, index) {
      deleteMedia(row).then((res) => {
        // console.log(res)
        if (res.code === 20000) {
          this.$notify({
            title: "提示",
            message: "删除成功",
            type: "success",
            duration: 2000,
          });
        }
        this.tableData.splice(index, 1);
      });
    },
    //搜索商品
    handleFilter() {
      this.listQuery.page = 1;
      this.getlist();
    },
    //添加商品
    sortChange(data) {
      const { prop, order } = data;
      if (prop === "id") {
        this.sortByID(order);
      }
    },
    sortByID(order) {
      if (order === "ascending") {
        this.listQuery.sort = "+id";
      } else {
        this.listQuery.sort = "-id";
      }
      this.handleFilter();
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        title: "",
        status: 1,
        price: 0,
        try: "",
        content: "",
        cover: "",
      };
    },
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    createData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024; // mock a id
          this.temp.author = "vue-element-admin";
          createMedia(this.temp).then(() => {
            console.log(this.temp);
            this.list.unshift(this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "Success",
              message: "Created Successfully",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row); // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp);
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    updateData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp);
          tempData.timestamp = +new Date(tempData.timestamp); // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateMedia(tempData).then(() => {
            const index = this.list.findIndex((v) => v.id === this.temp.id);
            this.list.splice(index, 1, this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "Success",
              message: "Update Successfully",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },
    getSortClass: function (key) {
      const sort = this.listQuery.sort;
      return sort === `+${key}` ? "ascending" : "descending";
    },
    handleUploadRemove(file, fileList) {
      console.log(file, fileList);
    },
    //上传图片
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },

    //计数器
    handleChange(value) {
      console.log(value);
    },
  },
  created() {
    this.getlist();
  },
};
</script>


<style scoped>
.main {
  margin: 20px;
}
.button {
  margin: 5px 0 0 5px;
}
.searchB {
  margin-right: 0;
}
.input {
  width: 200px;
  height: 36px;
}
.select {
  width: 90px;
  height: 36px;
  float: right;
}

.wocao {
  float: right;
  background: rgb(red, green, blue);
}
</style>