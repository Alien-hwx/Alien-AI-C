<template>
  <div class="page-container">
    <ai-table
      :table-actions="tableActions"
      :table-columns="tableColumns"
      :table-data="tableData"
      :table-action-column="tableActionColumn"
      :pagination="pagination"
      @refresh="handleRefresh"
    >
      <template #topActions>
        <!-- <el-button type="primary" icon="el-icon-plus" @click="handleCreate">新建</el-button> -->

        <!-- <el-button type="danger" icon="el-icon-delete" disabled>删除</el-button> -->
      </template>
      <template v-slot:rowActions="slotProps">
        <el-button icon="el-icon-edit" @click.stop="handleEdit(slotProps.row)">编辑</el-button>
        <el-button @click.stop="handleToggleEnable(slotProps.row)">
          {{ slotProps.row.state == 1 ? '停用' : '取消停用' }}
        </el-button>
      </template>
      <!-- <template #columns>
        <el-table-column type="selection" width="55" />
      </template> -->

      <template v-slot:tokens="slotProps">
        <el-tag v-if="slotProps.row.tokens == -1" type="success">无限制</el-tag>
        <el-tag v-else-if="slotProps.row.tokens < 5" type="warning">{{ slotProps.row.tokens }}</el-tag>
        <el-tag v-else type="success">{{ slotProps.row.tokens }}</el-tag>
      </template>
      <template v-slot:chatCount="slotProps">
        <el-tag v-if="slotProps.row.chatCount == -1" type="success">无限制</el-tag>
        <el-tag v-else-if="slotProps.row.chatCount < 5" type="warning">{{ slotProps.row.chatCount }}</el-tag>
        <el-tag v-else type="success">{{ slotProps.row.chatCount }}</el-tag>
      </template>
      <template v-slot:advancedChatCount="slotProps">
        <el-tag v-if="slotProps.row.advancedChatCount == -1" type="success">无限制</el-tag>
        <el-tag v-else-if="slotProps.row.advancedChatCount < 5" type="warning">{{ slotProps.row.advancedChatCount }}</el-tag>
        <el-tag v-else type="success">{{ slotProps.row.advancedChatCount }}</el-tag>
      </template>
      <template v-slot:drawCount="slotProps">
        <el-tag v-if="slotProps.row.drawCount == -1" type="success">无限制</el-tag>
        <el-tag v-else-if="slotProps.row.drawCount < 5" type="warning">{{ slotProps.row.drawCount }}</el-tag>
        <el-tag v-else type="success">{{ slotProps.row.drawCount }}</el-tag>
      </template>
      <template v-slot:state="slotProps">
        <el-tag v-if="slotProps.row.state == 1" type="success">正常</el-tag>
        <el-tag v-else type="danger">停用</el-tag>
      </template>

    </ai-table>

    <detail
      ref="detail"
      :show="showDetail"
      :member="detailModel"
      @changed="handleChanged"
      @close="handleCloseDetail"
    />
  </div>
</template>

<script>
// import { mapGetters } from 'vuex'
import AiTable from '@/components/Table'
import { getMembers } from '@/api/member'
import { enableUser } from '@/api/user'
import Detail from './detail'

export default {
  name: 'MemberIndex',
  components: { AiTable, Detail },
  data() {
    return {
      showDetail: false,
      tableActions: [{
        key: 'increase-tokens',
        label: '添加tokens',
        type: 'primary'
      }, {
        key: 'tokens-record',
        label: 'token记录'
      }, {
        key: 'message-record',
        label: '聊天记录'
      }, {
        key: 'edit',
        label: '编辑'
      }],
      tableColumns: [{
        label: '#',
        prop: 'id',
        width: 55
      }, {
        label: '昵称',
        prop: 'name'
      }, {
        label: '登录账号',
        prop: 'username'
      }, {
        label: '状态',
        prop: 'state',
        slot: 'state'
      }, {
        label: '剩余tokens',
        prop: 'tokens',
        slot: 'tokens',
        width: 120
      }, {
        label: '剩余聊天次数',
        prop: 'chatCount',
        slot: 'chatCount',
        width: 120
      }, {
        label: '剩余高级聊天次数',
        prop: 'advancedChatCount',
        slot: 'advancedChatCount',
        width: 120
      }, {
        label: '剩余绘画次数',
        prop: 'drawCount',
        slot: 'drawCount',
        width: 120
      }, {
        label: '创建时间',
        prop: 'createTime',
        width: 140
      // }, {
      //   label: '更新时间',
      //   prop: 'update_time'
      }],
      tableActionColumn: {
        width: 400
      },
      tableData: [],
      pagination: {
        total: 0

      },
      detailModel: {
        id: null,
        name: null,
        username: null,
        state: null,
        role: null,
        tokens: null,
        chatCount: null,
        advancedChatCount: null,
        drawCount: null
      }
    }
  },
  computed: {
  },
  created() {
    // if (!this.roles.includes('admin')) {
    //   this.currentRole = 'editorDashboard'
    // }
  },
  mounted() {
    this.reload()
  },
  methods: {
    reload() {
      getMembers().then(resp => {
        console.log('resp', resp)
        const users = resp.data || []
        this.tableData = users.map(user => {
          return {
            id: user.id,
            username: user.username,
            name: user.name,
            tokens: user.tokens,
            role: user.role,
            state: user.state,
            chatCount: user.chatCount,
            advancedChatCount: user.advancedChatCount,
            drawCount: user.drawCount,
            createTime: user.createTime,
            updateTime: user.updateTime
          }
        })
        this.pagination.total = this.tableData.length
      }).then(() => {
        console.log('this.showDetail', this.showDetail)
        if (this.showDetail) {
          const row = this.tableData.filter(r => r.id === this.detailModel.id)[0]
          console.log('row', row)
          if (row) {
            this.updateDetail(row)
          }
        }
      })
    },
    updateDetail(row) {
      console.log('updateDetail')
      this.detailModel = {
        id: row.id,
        name: row.name,
        username: row.username,
        state: row.state,
        role: row.role,
        tokens: row.tokens,
        chatCount: row.chatCount,
        advancedChatCount: row.advancedChatCount,
        drawCount: row.drawCount
      }
      this.$nextTick(() => {
        this.$refs.detail.reload()
      })
    },
    handleRefresh() {
      this.reload()
    },
    handleCreate() {
      this.$message.warning('开发中……')
    },
    handleEdit(row) {
      this.showDetail = true
      this.updateDetail(row)
    },
    handleCloseDetail() {
      this.showDetail = false
    },
    handleChanged() {
      this.reload()
    },
    handleToggleEnable(row) {
      console.log('handleToggleEnable')
      this.$confirm(
        row.state === 1 ? ('确定对' + row.username + '进行停用吗？') : ('确定对' + row.username + '取消停用吗？'),
        '操作确认',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(async() => {
        const enable = row.state === 2
        enableUser(row.id, enable).then(resp => {
          console.log('resp', resp)
          this.$message.success('操作成功！')
          this.reload()
        })
      })
    },
    getStateName(state) {
      return ({
        1: '正常',
        2: '禁用'
      })[state] || '未知'
    }
  }
}
</script>

<style lang="scss" scoped>
.dashboard-container {
  padding: 10px;
}

</style>
