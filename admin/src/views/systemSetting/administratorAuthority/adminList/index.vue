<template>
  <div class="divBox">
    <el-card class="box-card">
      <el-form inline size="small">
        <el-form-item>
          <el-select v-model="listPram.realName" placeholder="身份" clearable class="selWidth">
            <el-option
              v-for="item in roleList.list"
              :key="item.id"
              :label="item.roleName"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-select v-model="listPram.isDel" placeholder="删除状态" clearable class="selWidth">
            <el-option
              v-for="item in constants.deletedOrNormal"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-select v-model="listPram.status" placeholder="状态" clearable class="selWidth">
            <el-option
              v-for="item in constants.roleListStatus"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-input v-model="listPram.roleName" placeholder="姓名或者账号" clearable class="selWidth"/>
        </el-form-item>
        <el-form-item>
          <el-button size="mini" type="primary" @click.native="handleGetAdminList">查询</el-button>
        </el-form-item>
      </el-form>
      <el-form inline>
        <el-form-item>
          <el-button size="mini" type="primary" @click="handlerOpenEdit(0)">添加管理员</el-button>
        </el-form-item>
      </el-form>
      <el-table :data="listData.list" size="mini">
        <el-table-column label="姓名" prop="realName" />
        <el-table-column label="账号" prop="account" />
        <el-table-column label="身份" prop="realName">
          <template slot-scope="scope">
          <span v-for="item in roleList.list" :key="item.id">
            <el-tag v-if="scope.row.roles.includes(item.id)" type="info">{{ item.roleName }}</el-tag>
            <!--              <span>{{ scope.row }}</span>-->
          </span>
          </template>
        </el-table-column>
        <el-table-column label="最后登录时间" prop="lastTime">
          <template slot-scope="scope">
            <span>{{ scope.row.lastTime | filterEmpty }}</span>
          </template>
        </el-table-column>
        <el-table-column label="最后登录IP" prop="lastIp">
          <template slot-scope="scope">
            <span>{{ scope.row.lastIp | filterEmpty }}</span>
          </template>
        </el-table-column>
        <el-table-column label="状态" prop="status" width="100">
          <template slot-scope="scope">
            <span>{{ scope.row.status | filterShowOrHide }}</span>
          </template>
        </el-table-column>
        <el-table-column label="删除标记" prop="status" width="100">
          <template slot-scope="scope">
            <span>{{ scope.row.isDel | filterYesOrNo }}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <template v-if="scope.row.isDel">
              <span>-</span>
            </template>
            <template v-else>
              <el-button type="text" size="mini" @click="handlerOpenEdit(1,scope.row)">编辑</el-button>
              <el-button type="text" size="mini" @click="handlerOpenDel(scope.row)">删除</el-button>
            </template>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        :current-page="listPram.page"
        :page-sizes="constants.page.limit"
        :layout="constants.page.layout"
        :total="listData.total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </el-card>
    <el-dialog
      :visible.sync="editDialogConfig.visible"
      :title="editDialogConfig.isCreate === 0? '创建身份':'编辑身份'"
      destroy-on-close
      :close-on-click-modal="false"
    >
      <edit
        v-if="editDialogConfig.visible"
        :is-create="editDialogConfig.isCreate"
        :edit-data="editDialogConfig.editData"
        @hideEditDialog="hideEditDialog"
      />
    </el-dialog>
  </div>
</template>

<script>
import * as systemAdminApi from '@/api/systemadmin.js'
import * as roleApi from '@/api/role.js'
import * as constants from '@/utils/constants.js'
import edit from './edit'
export default {
  // name: "index"
  components: { edit },
  data() {
    return {
      constants,
      listData: { list: [] },
      listPram: {
        account: null,
        addTime: null,
        isDel: null, // false=正常，true=已删除，数据库逻辑删除
        lastIp: null,
        lastTime: null,
        level: null,
        loginCount: null,
        realName: null,
        roles: null,
        status: null,
        page: 1,
        limit: constants.page.limit[1]
      },
      roleList: [],
      menuList: [],
      editDialogConfig: {
        visible: false,
        isCreate: 0, // 0=创建，1=编辑
        editData: {}
      }
    }
  },
  mounted() {
    this.handleGetAdminList()
    this.handleGetRoleList()
  },
  methods: {
    handleSizeChange(val) {
      this.listPram.limit = val
      this.handleGetRoleList(this.listPram)
    },
    handleCurrentChange(val) {
      this.listPram.page = val
      this.handleGetRoleList(this.listPram)
    },
    handleGetRoleList() {
      const _pram = {
        page: 1,
        limit: constants.page.limit[4]
      }
      roleApi.getRoleList(_pram).then(data => {
        this.roleList = data
      })
    },
    handlerOpenDel(rowData) {
      this.$confirm('确认删除当前数据').then(() => {
        const _pram = { id: rowData.id, isDel: 1, roles: Array.of(rowData.roles) }
        systemAdminApi.adminUpdate(_pram).then(data => {
          this.$message.success('删除数据成功')
          this.handleGetAdminList()
        })
      })
    },
    handleGetAdminList() {
      systemAdminApi.adminList(this.listPram).then(data => {
        this.listData = data
        // this.handlerGetMenuList()
      })
    },
    handlerOpenEdit(isCreate, editDate) {
      this.editDialogConfig.editData = editDate
      this.editDialogConfig.isCreate = isCreate
      this.editDialogConfig.visible = true
    },
    handlerGetMenuList() { // 获取菜单全部数据后做menu翻译使用
      systemAdminApi.listCategroy({ page: 1, limit: 999, type: 5 }).then(data => {
        this.menuList = data.list
        this.listData.list.forEach(item => {
          const _muneText = []
          const menuids = item.rules.split(',')
          menuids.map(muid => {
            this.menuList.filter(menu => {
              if (menu.id == muid) {
                _muneText.push(menu.name)
              }
            })
          })
          item.rulesView = _muneText.join(',')
          this.$set(item, 'rulesViews', item.rulesView)
        })
      })
    },
    hideEditDialog() {
      this.editDialogConfig.visible = false
      this.handleGetAdminList()
    }
  }
}
</script>

<style scoped>

</style>
