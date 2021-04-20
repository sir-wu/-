<template>
    <div>
       
  <el-row class="main" :gutter="20">
      <el-col :span="6">
  <el-button class="button" type="primary"> <i class="el-icon-edit"></i>新增图文</el-button>
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
  <el-input v-model="input" placeholder="请输入内容" class="input"></el-input>
 <el-button class="searchB" type="primary"> <i class="el-icon-search"></i>搜索</el-button>
 </el-col>
</el-row>

 <el-table :data="tableData" style="width: 100%" border>
      <el-table-column sortable prop="id" label="ID" width="100" align="center">
      </el-table-column>
      <el-table-column prop="content" label="图文内容" width="500">
      </el-table-column>
      <el-table-column
        prop="sub_count"
        label="订阅量"
        width="100"
        align="center"
      >
      </el-table-column>
      <el-table-column prop="status" label="状态" width="100" align="center">
          <template >
              <el-button
                size="mini"
                type="success" 
                plain 
              >
                已上架
              </el-button>
          </template>
      </el-table-column>
      <el-table-column prop="created_time" label="创建时间" align="center">
      </el-table-column>
      <el-table-column prop="try" label="操作" align="center">
          <template >
              <el-button
                size="mini"
                type="primary" 
              >
                编辑
              </el-button>
              <el-button
                size="mini"
                type="primary" 
                >
                上架
              </el-button>
              <el-button
                size="mini"
                type="danger" 
                >
                删除
              </el-button>
          </template>
      </el-table-column>
    </el-table>

    <div class="block">
      <el-pagination
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage1"
        :page-sizes="[20, 40, 60, 80, 100]"
        :page-size="20"
        layout="total, sizes, prev, pager, next, jumper"
        :total="100"
      >
      </el-pagination>
    </div>
    </div>
</template>

<script>
export default {
    data() {
      return {
        options: [{
          value: '选项1',
          label: '已上架'
        }, {
          value: '选项2',
          label: '已下架'
        }],
        value: '',
        input: '',
        currentPage1: 1,
      currentPage2: 2,
      currentPage3: 3,
      currentPage4: 4,
      currentPage5: 5,
      tableData: [],
      formInline: {
        user: "",
        region: "",
      },
      }
    },
    methods: {
    onSubmit() {
      console.log("submit!");
    },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
    },
  },
  
}
</script>


<style scoped>
.main{
    margin: 20px;
}
.button{
    margin:5px 0 0 5px;
}
.searchB{
    margin-right: 0;
}
.input{
    width: 200px;
    height: 36px;
}
.select{
    width: 90px;
    height: 36px;
    float: right;
}

.wocao{
    float: right;
}


</style>