<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('generator:gafferinfo:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('generator:gafferinfo:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
<!--      <el-table-column-->
<!--        prop="id"-->
<!--        header-align="center"-->
<!--        align="center"-->
<!--        label="">-->
<!--      </el-table-column>-->
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="姓名">
      </el-table-column>
      <el-table-column
        prop="sex"
        header-align="center"
        align="center"
        label="性别">
      </el-table-column>
      <el-table-column
        prop="age"
        header-align="center"
        align="center"
        label="年龄">
      </el-table-column>
      <el-table-column
        prop="telephone"
        header-align="center"
        align="center"
        label="电话">
      </el-table-column>
      <el-table-column
        prop="guardianName"
        header-align="center"
        align="center"
        label="监护人姓名">
      </el-table-column>
      <el-table-column
        prop="guardianTelephone"
        header-align="center"
        align="center"
        label="监护人电话">
      </el-table-column>
      <el-table-column
        prop="address"
        header-align="center"
        align="center"
        label="家庭住址">
      </el-table-column>
<!--      <el-table-column-->
<!--        prop="pic"-->
<!--        header-align="center"-->
<!--        align="center"-->
<!--        label="相片">-->
<!--      </el-table-column>-->
      <el-table-column
        align="center"
        label="相片">
        <template slot-scope="scope">
          <el-image
            style="width: 80px; height: 80px"
            :src="scope.row.pic"
            :fit="'fill'"></el-image>
        </template>
      </el-table-column>
      <el-table-column
        prop="walletAmount"
        header-align="center"
        align="center"
        label="钱包余额">
      </el-table-column>
<!--      <el-table-column-->
<!--        prop="healthRecord"-->
<!--        header-align="center"-->
<!--        align="center"-->
<!--        label="健康档案">-->
<!--      </el-table-column>-->
      <el-table-column
        prop="user.username"
        header-align="center"
        align="center"
        label="操作人">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="200"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="showHealthRecord(scope.row)">健康档案</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <edit-health-file v-if="dialogHealthVisible" ref="editHealthFile" @refreshDataList="getDataList" ></edit-health-file>
  </div>
</template>

<script>
  import AddOrUpdate from './gafferinfo-add-or-update'
  import editHealthFile from './gafferinfo-edit-health-file.vue'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        dialogHealthVisible: false
      }
    },
    components: {
      AddOrUpdate,
      editHealthFile
    },
    activated () {
      this.getDataList()
    },
    methods: {
      showHealthRecord(data) {
        console.log('data',data);
        this.dialogHealthVisible = true;
        this.$nextTick(() => {
          this.$refs.editHealthFile.init(data.id)
        })
      },
      // 获取数据列表
      getDataList () {
        let that = this;
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/gafferinfo/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
            that.getUserInfo();
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      getUserInfo() {
        let dataList = this.dataList
        let userIdList = []
        dataList.forEach(data => {
          userIdList.push(data.operationUserId)
        })
        this.$http({
          url: this.$http.adornUrl('/sys/user/listByIds'),
          method: 'get',
          params: this.$http.adornParams({
            'userIdList': userIdList.map(item => item).join(',')
          })
        }).then(({data}) => {
          if (data && data.code == 0) {
            let userList = data.userList;
            this.dataList.forEach(item => {
              userList.forEach(user => {
                if (item.operationUserId == user.userId) {
                  this.$set(item,'user',user);
                  return;
                }
              });
            });
          }
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/generator/gafferinfo/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
