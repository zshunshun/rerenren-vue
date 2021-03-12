<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="员工关键词" clearable></el-input>
      </el-form-item>
      <el-form-item label=" 年月:" prop="starDate">
        <!--      <el-input v-model="dataForm.starDate" placeholder="评级所属年月"></el-input>-->
        <el-date-picker
          v-model="dataForm.recordDate"
          type="month"
          placeholder="选择月">
        </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
<!--        <el-button v-if="isAuth('generator:empsalaryrecord:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>-->
<!--        <el-button v-if="isAuth('generator:empsalaryrecord:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>-->
        <el-button v-if="isAuth('generator:empsalaryrecord:delete')" type="success" @click="generateHandle()" >生成上月工资</el-button>
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
      <el-table-column
        prop="emp.name"
        header-align="center"
        align="center"
        label="员工">
      </el-table-column>
      <el-table-column
        prop="amount"
        header-align="center"
        align="center"
        label="本月基本工资">
      </el-table-column>
      <el-table-column
        prop="starAmount"
        header-align="center"
        align="center"
        label="考评工资">
      </el-table-column>
      <el-table-column
        prop="attendanceAmount"
        header-align="center"
        align="center"
        label="考勤工资">
      </el-table-column>
      <el-table-column
        prop="totalAmount"
        header-align="center"
        align="center"
        label="总工资">
      </el-table-column>
      <el-table-column
        prop="user.username"
        header-align="center"
        align="center"
        label="操作人">
      </el-table-column>
      <el-table-column
        prop="recordDate"
        header-align="center"
        align="center"
        label="记录所属年月">
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
        width="150"
        label="操作">
<!--        <template slot-scope="scope">-->
<!--&lt;!&ndash;          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>&ndash;&gt;-->
<!--&lt;!&ndash;          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>&ndash;&gt;-->
<!--        </template>-->
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
  </div>
</template>

<script>
  import AddOrUpdate from './empsalaryrecord-add-or-update';
  import { formatDate } from '@/utils/dateUtils';
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
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        let recordDate = this.dataForm.recordDate;
        this.$http({
          url: this.$http.adornUrl('/generator/empsalaryrecord/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key,
            'recordDate': (recordDate && typeof(recordDate) != 'string') ? formatDate(recordDate,'yyyy-MM') : recordDate,
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
            this.fileEmpName();
            this.fileUserName();
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 填充员工信息
      fileEmpName() {
        let dataList = this.dataList;
        let empIdList = dataList.map(item => item.empId);
        this.$http({
          url: this.$http.adornUrl('/generator/empinfo/listByIds'),
          method: 'get',
          params: this.$http.adornParams({
            'empIdList': empIdList.join(',')
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            dataList.forEach(item => {
              data.empList.forEach(emp => {
                if (item.empId === emp.id) {
                  this.$set(item,'emp',emp);
                  return;
                }
              })
            })
          }
        })
      },
      // 填充操作人信息
      fileUserName() {
        let dataList = this.dataList;
        let operationUserId = dataList.map(item => item.operationUserId);
        this.$http({
          url: this.$http.adornUrl('/sys/user/listByIds'),
          method: 'get',
          params: this.$http.adornParams({
            'userIdList': operationUserId.join(',')
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            dataList.forEach(item => {
              data.userList.forEach(user => {
                if (item.operationUserId === user.userId) {
                  this.$set(item,'user',user);
                  return;
                }
              })
            })
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
            url: this.$http.adornUrl('/generator/empsalaryrecord/delete'),
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
      },
      generateHandle () {
        this.$http({
          url: this.$http.adornUrl('/generator/empsalaryrecord/generateForLastMonth'),
          method: 'post',
          data: this.$http.adornData("", false)
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
      }
    }
  }
</script>
