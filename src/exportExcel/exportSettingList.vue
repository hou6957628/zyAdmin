<template>
  <div class="content">
    <el-breadcrumb class="fs-16" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/riskControl' }">导出配置</el-breadcrumb-item>
    </el-breadcrumb>
    <div class="operationContent">
      <el-button class="upLoadBtn" @click="toAddProduct()" type="primary">创建导出配置<i
        class="el-icon-upload el-icon-circle-plus"></i></el-button>
      <el-input @click="searchProduct" class="searchContent"
                placeholder="导出文件名称"
                v-model="excelName"
                clearable>
        <el-button id="searchBtn" @click="searchContent(excelName)" slot="append" icon="el-icon-search">查询</el-button>
      </el-input>
    </div>
    <template>
      <el-table
        :data="tableData"
        border
        highlight-current-row
        style="width: 100%">
        <el-table-column
          fixed
          prop="excelName"
          label="导出报表名称"
          width="200">
        </el-table-column>
        <el-table-column
          prop="colomnNames"
          label="列名"
          width="170">
        </el-table-column>
        <el-table-column
          prop="excelColomnNames"
          label="excel列名"
          width="250">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="创建时间"
          width="170">
        </el-table-column>
        <el-table-column
          prop="updateDate"
          label="更新时间"
          width="170">
        </el-table-column>
        <el-table-column
          prop="enabled"
          label="状态"
          width="80">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.enable == true ? 'primary' : 'danger'"
              disable-transitions>{{scope.row.enable == true ? '启用' : '禁用'}}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="200">
          <template slot-scope="scope">
            <el-button @click="editProduct(scope.row)" type="text" size="medium">编辑</el-button>
            <el-button @click="exportExcel(scope.row)" type="text" size="medium">导出excel</el-button>
            <!--<el-button v-if="scope.row.enable" @click="obtainedProduct(scope.row)" type="text" size="medium">停用</el-button>-->
            <!--<el-button v-if="!scope.row.enable" @click="obtainedCustomerTag(scope.row)" type="text" size="medium">启用</el-button>-->
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
    methods: {
      //条件查询风控流程
      searchContent(data) {
        this.getProductList(1, 20, this.finProduct);
      },
      //每页显示多少条
      handleSizeChange(val) {
        this.getProductList(this.pageNum, val, this.finProduct, this.finProduct);
        this.nowPageSizes = val;
      },
      //翻页
      handleCurrentChange(val) {
        this.getProductList(val, this.nowPageSizes, this.finProduct, this.finProduct);
      },
      //创建额度流程
      toAddProduct() {
        this.$router.push({
          path: `/addExcelSetting`,
        });
      },
      /**
       * 用户标签列表接口
       * @param data1 查询第几页
       * @param data2 每页显示多少条数据
       * @param data3 标签名称
       * @param data4 分类名称
       */
      getProductList(data1, data2, data3) {
        axios({
          method: "GET",
          url: "http://" + this.baseUrl + "/export/admin/setting/list",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            pageNum: data1,
            pageSize: data2,
            flowName: data3,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.tableData = res.data.body.list;
            this.proTotal = res.data.body.total;
            this.pageSize = res.data.body.pageSize;
            this.pageNum = res.data.body.pageNum;
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //查询产品接口
      searchProduct() {
        this.getProductList(1, 20, this.finProduct, this.finProduct, null);
      },
      //查询产品接口
      exportExcel(row) {
       var url = "http://" + this.baseUrl + "/export/admin/excel/export?id="+row.id;
       window.location.href = url;
      },
      //编辑产品接口
      editProduct(row) {
        var id = row.id;
        this.$router.push({
          path: `/editExportSetting/${id}`,
        });
      },
      //配置产品接口
      configureProduct(row) {
        var id = row.id;
        this.$router.push({
          path: `/configAmountFlow/${id}`,
        });
      },
      //提示启用停用规则接口
      obtainedProduct(row) {
        axios({
          method: "post",
          url: "http://" + this.baseUrl + "/quota/admin/amountFlow/checkRef",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            flowId: row.id,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            if (res.data.body == "true") {
              this.$alert('该额度流程在某应用（APP）引用，不可停用', '提示', {
                confirmButtonText: '确定',
                center: true,
                type: 'warning'
              });
            } else {
              this.$confirm('是否确认停用此额度流程?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
              }).then(() => {
                this.obtainedCustomerTag(row);
              }).catch(() => {
                this.$message({
                  type: 'info',
                  message: '已取消操作'
                });
              });
            }
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
      //确认启用停用规则接口
      obtainedCustomerTag(row) {
        axios({
          method: "post",
          url: "http://" + this.baseUrl + "/quota/admin/amountFlow/deleteOrUpdate",
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Authorization': localStorage.token
          },
          params: {
            id: row.id,
            enabled: !row.enabled,
          }
        }).then((res) => {
          if (res.data.msgCd == 'ZYCASH-200') {
            this.$message({
              message: '操作成功',
              type: 'success'
            });
            this.getProductList(1, 20, null);
          } else {
            this.$message.error(res.data.msgInfo);
          }
        })
      },
    },
    mounted: function () {
      this.getProductList(1, 20, null);
    },
    data() {
      return {
        tableData: [],
        excelName: '',
        pageNum: null,
        proTotal: null,
        pageSize: null,
        pageSizes: [20, 30, 50],
        nowPageSizes: 20,
        centerDialogVisible: false,
        electDataEnabled: [
          {key: 1, Id: '启用'},
          {key: 0, Id: '不启用'},
        ],
        ruleForm: {
          copyId: '',
          ruleName: '',
          ruleDetail: '',
          enabled: 1,
        },
        rules: {
          enabled: [
            {required: true, message: '请选择是否启用', trigger: 'change'}
          ],
        },
      }
    }
  }
</script>

<style scoped>
  .select {
    margin-left: 20px;
  }

  .content {
    padding-left: 200px;
    padding-top: 80px;
  }

  .fs-16 {
    font-size: 20px;
    margin-top: 10px;
    margin-bottom: 20px;
  }

  .operationContent {
    text-align: left;
    margin: 25px 30px 15px 0;
  }

  .operationContent .upLoadBtn {

  }

  .operationContent .searchContent {
    margin-left: 20px;
    width: 300px;
    margin-right: 30px;
  }

  .paginationBox {
    margin-top: 20px;
    font-size: 18px;
    margin-bottom: 40px;
  }
</style>
