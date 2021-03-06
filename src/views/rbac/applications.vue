<template>
  <div class="app-container">
    <el-button type="danger" @click="handleAddApplication">New Application</el-button>
    <el-table
      v-loading="listLoading"
      :data="applications"
      border
      fit
      highlight-current-row
      style="width: 100%;margin-top:30px"
    >
      <el-table-column align="center" label="System ID" width="100px">
        <template slot-scope="scope">{{ scope.row.id }}</template>
      </el-table-column>

      <el-table-column align="center" min-width="100px" label="Application Name">
        <template slot-scope="scope">{{ scope.row.name }}</template>
      </el-table-column>
      <el-table-column align="center" label="Roles" width="120">
        <template slot-scope="scope">
          <router-link :to="'/rbac/roles/'+scope.row.id" class="link-type">
            <el-button type="success" size="small" icon="el-icon-document">Roles</el-button>
          </router-link>
        </template>
      </el-table-column>
      <el-table-column align="center" label="Scopes" width="120">
        <template slot-scope="scope">
          <router-link :to="'/rbac/scopes/'+scope.row.id" class="link-type">
            <el-button type="primary" size="small" icon="el-icon-document">Scopes</el-button>
          </router-link>
        </template>
      </el-table-column>
      <el-table-column align="center" label="Scopes" width="120">
        <template slot-scope="scope">
          <router-link :to="'/rbac/app-i18n-names/'+scope.row.id" class="link-type">
            <el-button type="primary" size="small" icon="el-icon-document">App I18N Names</el-button>
          </router-link>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog :visible.sync="newApplicationVisible" :title="'New Application'">
      <el-form :model="application" label-width="80px" label-position="left">
        <el-form-item label="Name">
          <el-input v-model="application.name" placeholder="Role Name" />
        </el-form-item>
      </el-form>
      <div style="text-align:right;">
        <el-button type="danger" @click="newApplicationVisible=false">Cancel</el-button>
        <el-button type="primary" @click="confirmNewApplication">Confirm</el-button>
      </div>
    </el-dialog>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getApplicationList"
    />
  </div>
</template>

<script>
import { deepClone } from '@/utils'
import { getApplications, createApplication } from '@/api/rbac'
import Pagination from '@/components/Pagination' // Secondary package based on el-pagination

const defaultApplication = {
  name: '',
  defaultRole: null
}

export default {
  name: 'Applications',
  components: { Pagination },
  data() {
    return {
      application: Object.assign({}, defaultApplication),
      applications: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20
      },
      newApplicationVisible: false
    }
  },
  watch: {
    // call again the method if the route changes
    $route: 'getApplicationList'
  },
  created() {
    this.getApplicationList()
  },
  methods: {
    getApplicationList() {
      this.listLoading = true
      getApplications().then((response) => {
        console.log(response.body.applications)
        this.applications = response.body.applications
        this.total = this.applications.length
        this.listLoading = false
      })
    },
    handleAddApplication() {
      this.application = Object.assign({}, defaultApplication)
      this.newApplicationVisible = true
    },
    handleEditApplication(scope) {
      this.application = deepClone(scope.row)
      this.editApplicationVisible = true
    },
    async confirmNewApplication() {
      var data = {
        name: this.application.name
      }
      await createApplication(data)
      this.newApplicationVisible = false
      this.getApplicationList()
    }
  }
}
</script>

<style scoped>
.edit-input {
  padding-right: 100px;
}
.cancel-btn {
  position: absolute;
  right: 15px;
  top: 10px;
}
</style>
