<template>
<div>
  
    <el-card shadow="never">
        <div style="display: flex;align-items: flex-start;">
            <img :src="detail.cover" style="width: 200px;height: 100px;margin-right: 20px;">
          
           <div style="flex: 1;">
               <div  style="display: flex;justify-content: space-between;">
                   <h4 style="margin-top: 5px;margin-bottom: 0;">{{detail.title}}</h4>
                   <small style="color: #bbbbbb;">{{ detail.isend ? '已完结' : '连载中' }}</small>
               </div>
          
            <p style="margin: 8px 0;">
            <small style="color: #bbbbbb;">{{ detail.try }}</small>
           </p>

           <p>
         <small style="color: red;">￥{{ detail.price }}</small>
         </p>

         <el-button-group >
             <el-button size="small" type="warning" @click="changeDetailStatus">
                {{ detail.status ? '下架' : '上架' }}
            </el-button>
        <el-button size="small" type="default" @click="changeDetailIsend">设为{{ detail.isend ? '连载中' : '已完结' }}</el-button>
          </el-button-group>
           </div> 
        </div>

    </el-card>


    <div>
  <el-row class="main" :gutter="20">
      <el-col :span="6">
  <el-button class="button" type="primary" @click="addCourse" > <i class="el-icon-edit"></i>新增目录</el-button>
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
      <el-table-column  label="单品内容" width="500">
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
          <!-- <el-button v-if="row.status == 0" size="mini" type="success" @click="handleModifyStatus(row,1)">
            上架
          </el-button>
          <el-button v-else-if="row.status == 1" size="mini" @click="handleModifyStatus(row,0)">
            下架
          </el-button> -->
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


    <!-- 增加目录区域 -->

   <choose-course ref="chooseCourse"></choose-course>
    
       
    </div>
    </div>
</template>

<script>
let id=0
import waves from '@/directive/waves' 
import {
  fetchDetail,
  fetchDetailCourse,
  deleteMedia,
  createMedia,
  updateMedia,
} from "../../../api/column";
import Tinymce from "@/components/Tinymce";
import chooseCourse from '@/components/chooseCourse/index.vue'

let defaultListQuery = {
  status: undefined,
  title: undefined,
  page: 1, // 当前是第几页
  limit: 10, // 每页5条数据
};

const statusOptions = {
  0: "已下架",
  1: "已上架",
};
export default {
  components: { Tinymce ,chooseCourse},
//   beforeRouteEnter(to, from, next) {
//             id = to.query.id
//             next()
//         },
//把状态上架下架过滤出来
  directives: {
            waves
        },
    
   beforeRouteEnter(to, from, next) {
       console.log(to.query.id)
            id = to.query.id
            next()
        },
  filters: {
    statusFilter(status) {
      return statusOptions[status];
    },
  },
  data() {
    return {
      fileList: [
        {
          name: "food.mp3",
          url:
            "https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100",
        },
        {
          name: "food2.jpeg",
          url:
            "https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100",
        },
      ],
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
      detial:{

      },
     
     detail: {
                    content: "",
                    cover: "",
                    created_time: "",
                    id: 0,
                    isend: 0,
                    price: 0,
                    status: 1,
                    sub_count: 0,
                    title: "",
                    try: "",
                    updated_time: ""
                },

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
      shangxiajia:true,
      wanjie:true
    };
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

      let result = await fetchDetailCourse(this.listQuery);
      console.log(this.listQuery);
      console.log(result);
      if (result.code === 20000) {
        this.tableData = result.data.items;
        this.total = result.data.total;
        this.list = result.data.items;
        // console.log(this.total)
        this.listLoading = false;
      }
    },
    //获取每个专栏对应的数据
getdata(){
     console.log(this.id)
   fetchDetail({id}).then(res=>{
            this.detail=res.data
     })
        //  console.log("........................................")
        //  console.log( "..",result)
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
   //新增目录
   addCourse(){
                this.$refs.chooseCourse.open((val)=>{
                    this.list = [...this.list,...val]
                },50)
                // this.$refs.chooseCourse.open()
            },
    //改变那两个按钮
    changeDetailStatus(){
                this.detail.status = this.detail.status ? 0 : 1
            },
            changeDetailIsend(){
                this.detail.isend = this.detail.isend ? 0 : 1
            },
  },
  created() {
    this.getlist();
    this.getdata()
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