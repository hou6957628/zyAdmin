<template>
  <div>
    <h3>用户催收记录</h3>
    <template>
      <el-table
        :data="collection"
        border
        highlight-current-row
        style="width: 98%;margin-top: 20px;">
        <el-table-column
          fixed
          prop="id"
          label="编号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="collectorName"
          label="催收人员"
          width="120">
        </el-table-column>
        <el-table-column
          prop="collectionLabelName"
          label="类型"
          width="150">
        </el-table-column>
        <el-table-column
          prop="callTypeName"
          label="通话类型"
          width="150">
        </el-table-column>
        <el-table-column
          prop="memo"
          label="说明"
          width="400">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="时间"
          width="200">
        </el-table-column>
        <el-table-column
          label="操作"
          width="120">
          <template slot-scope="scope">
            <el-button @click="detailProduct(scope.row)" type="text" size="medium">详情</el-button>
          </template>
        </el-table-column>
      </el-table>
    </template>
    <div class="block">
      <el-pagination class="paginationBox"
                     @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :unique-opened="true"
                     :current-page="pageNum"
                     :page-sizes="pageSizes"
                     :page-size="pageSize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="proTotal">
      </el-pagination>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    data() {
      return {
        collection: [],
        pageNum: null,
        proTotal:null,
        pageSize:null,
        pageSizes:[30,50,80],
        nowPageSizes:30,
      };
    },
    methods: {
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum,val,this.id);
        this.nowPageSizes=val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val,this.nowPageSizes,this.id);
      },
      //详情
      detailProduct(row){
        let id=row.id;
        this.$router.push({
          path: `/editCollectRecord/${id}`,
        });
      },
      //用户催收记录
      getCollection(data1,data2,data3) {
        axios({
          method: "POST",
          url:"http://"+this.baseUrl+"/user_center/admin/collection/get",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            pageNum: data1,
            pageSize: data2,
            id: data3,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.collection = res.data.body.list;
            this.proTotal=res.data.body.total;
            this.pageSize=res.data.body.pageSize;
            this.pageNum=res.data.body.pageNum;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted: function () {
      this.id=this.$route.params.id;
      this.getCollection(1,30,this.id);
    }
  }
</script>

<style scoped>
  .listBox{
    display: inline-block;
    float: left;
    padding: 10px 20px;
    margin: 0 5px;
    font-size: 17px;
    color: #000;
    background-color: #dedede;
    border: 1px solid #b0b0b0;
    border-radius: 10px;
    cursor: pointer;
  }
  .content {
    padding-left: 200px;
    padding-top: 80px;
  }
  .listContent{
    clear: both;
    height: 80px;
  }

  .paginationBox{
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }
</style>
