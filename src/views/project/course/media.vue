<template>
    <div>
       
  <el-row class="main" :gutter="20">
      <el-col :span="6">
  <el-button class="button" type="primary" @click="handleCreate"> <i class="el-icon-edit"></i>新增图文</el-button>
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
      <el-table-column prop="content" label="图文内容" width="500">
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
          <el-button type="primary" size="mini">
            编辑
          </el-button>
          <el-button v-if="row.status == 0" size="mini" type="success" @click="handleModifyStatus(row,1)">
            上架
          </el-button>
          <el-button v-if="row.status == 1" size="mini" @click="handleModifyStatus(row,0)">
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

        <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <el-form-item label="Type" prop="type">
          <el-select v-model="temp.type" class="filter-item" placeholder="Please select">
            <el-option v-for="item in calendarTypeOptions" :key="item.key" :label="item.display_name" :value="item.key" />
          </el-select>
        </el-form-item>
        <el-form-item label="Date" prop="timestamp">
          <el-date-picker v-model="temp.timestamp" type="datetime" placeholder="Please pick a date" />
        </el-form-item>
        <el-form-item label="Title" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="Status">
          <el-select v-model="temp.status" class="filter-item" placeholder="Please select">
            <el-option v-for="item in statusOptions" :key="item" :label="item" :value="item" />
          </el-select>
        </el-form-item>
        <el-form-item label="Imp">
          <el-rate v-model="temp.importance" :colors="['#99A9BF', '#F7BA2A', '#FF9900']" :max="3" style="margin-top:8px;" />
        </el-form-item>
        <el-form-item label="Remark">
          <el-input v-model="temp.remark" :autosize="{ minRows: 2, maxRows: 4}" type="textarea" placeholder="Please input" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          Cancel
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          Confirm
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>
    </div>
</template>

<script>
import { fetchList ,deleteMedia} from "../../../api/media";
let defaultListQuery = {
  page: 1, // 当前是第几页
  limit: 10, // 每页5条数据
  title: undefined,
};
const statusOptions = {
        0:"已下架",
        1:"已上架"
    }
export default {
  
  data() {
    return {
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
      temp: {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        type: '',
        status: 'published'
      },
      dialogPvVisible: false,
      dialogStatus: '',
      dialogFormVisible: false,
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
    };
  },
  //把状态上架下架过滤出来
  filters: {
            statusFilter(status) {
                return statusOptions[status]
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
        // console.log(this.total)
        this.listLoading = false;
      }
    },

    //控制产品上架下架
    handleModifyStatus(row, status) {
                this.$message({
                    message: '操作Success',
                    type: 'success'
                })
                row.status = status
            },
    //删除商品
    handleDelete(row,index){
      deleteMedia(row).then(res=>{
       // console.log(res)
        if(res.code===20000){
          this.$notify({
                        title: '提示',
                        message: '删除成功',
                        type: 'success',
                        duration: 2000
                    })
        }
        this.tableData.splice(index, 1)
      })
    },
    //搜索商品
    handleFilter() {
                this.listQuery.page = 1
                this.getlist()
            },
    //添加商品
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        status: 'published',
        type: ''
      }
    },
     sortChange(data) {
      const { prop, order } = data
      if (prop === 'id') {
        this.sortByID(order)
      }
    },
    sortByID(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
     createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          this.temp.author = 'vue-element-admin'
          createArticle(this.temp).then(() => {
            this.list.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
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