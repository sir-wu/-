<template>
    <div>
       
  <el-row class="main" :gutter="20">
      <el-col :span="6">
  <el-dropdown>
  <el-button type="danger">
    黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
  </el-button>
  <el-dropdown-menu slot="dropdown">
    <el-dropdown-item>禁止评论</el-dropdown-item>
    <el-dropdown-item>禁止访问</el-dropdown-item>
  </el-dropdown-menu>
</el-dropdown>
</el-col>

    <el-col :span="6" class="wocao">
  <el-input   placeholder="用户名" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
 <el-button  v-model="listQuery.title" class="searchB" type="primary" @click="handleFilter"> <i class="el-icon-search"></i>搜索</el-button>
 </el-col>
</el-row>


 <el-table
     border
    ref="multipleTable"
    :data="tableData"
    tooltip-effect="dark"
    style="width: 100%"
    @selection-change="handleSelectionChange">
    <el-table-column
      type="selection"
      width="55">
    </el-table-column>
    <el-table-column
      label="用户"
      width="500px">
      <template slot-scope="{row}">
        <div style="display: flex;align-items: center;">
            <img :src="row.user.avatar" style="width: 50px;height: 50px;margin-right: 10px;border-radius: 100%;">
            <span>{{ row.user.username }}</span>
        </div>
    </template>
    </el-table-column>
    <el-table-column
      prop="name"
      label="消费总额"
      width="120">
      <template slot-scope="{row}">
        <span>{{ row.total_consume }}</span>
    </template>
    </el-table-column>
    <el-table-column
      prop="address"
      label="创建时间"
      show-overflow-tooltip>
      <template slot-scope="{row}">
        <span>{{ row.created_time }}</span>
    </template>
    </el-table-column>
    <el-table-column
      prop="address"
      label="操作"
      show-overflow-tooltip>

      <el-button type="primary" size="mini">
            详情
    </el-button>
    <el-button type="success" size="mini">
            联系用户
     </el-button>

<el-button type="danger" size="mini">
    黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
  </el-button>

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
        </div>


       

</template>

<script>
import {
  fetchList,
  fetchUserDetail,
  fetchUserCourse,
  fetchUserColumn,
  fetchUserOrder,
  fetchUserHistory,
  fetchUserComment,
  changeCommentStatus,
  changeAccessStatus,
  fetchRole,
  login,
  getInfo,
  logout
} from "../../../api/user";
import Tinymce from "@/components/Tinymce";




export default {
  components: { Tinymce },
  data() {
    return{
        listQuery: {
            page:1,
            limit:20,
            title:""
        },
        listLoading: true,
        multipleSelection: [],
        tableData:[],
        total:0,
      }
  },
  methods: {
//获取数据
    async getlist() {
      // console.log(this.defaultListQuery)
      this.listLoading = true;
      
      let result = await fetchList(this.listQuery);
    //   console.log(this.listQuery);
    //   console.log(result);
      if (result.code === 20000) {
        this.tableData = result.data.items;
        this.total = result.data.total;
        // console.log(this.total)
        this.listLoading = false;
      }
    },
//全选反选按钮
     handleSelectionChange(val) {
        this.multipleSelection = val;
      },
    //搜索商品
    handleFilter() {
      this.listQuery.page = 1;
      this.getlist();
    },
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