<template>
  <div class="group block event-table">
    <el-table
      :data='tableData'
      stripe
      v-loading='loading'
      element-loading-spinner='el-icon-loading'
      :element-loading-text="$t('common.loading')"
      element-loading-background='rgba(256,256,256,0.8)'
    >
      <el-table-column
        prop='blockNumber'
        :label="$t('tableCont.blockNumber')"
        width=150
      ></el-table-column>
      <el-table-column
        prop='transCount'
        :label="$t('tableCont.transCount')"
        width=100
      ></el-table-column>
      <el-table-column
        prop='blockTimestamp'
        :label="$t('tableCont.timestamp')"
        width=230
      ></el-table-column>
      <el-table-column
        :label="$t('tableCont.pkHash')"
      >
      <template  slot-scope="scope">
        <i class='el-icon-copy-document' style='margin-right:5px;cursor:pointer' v-clipboard:copy='scope.row.pkHash' v-clipboard:success='onCopy'></i>
        <a :title='scope.row.logMsg' style='cursor:pointer' >{{scope.row.pkHash}}</a>
      </template>
      </el-table-column>
      <el-table-column
        :label="$t('common.action')"
        width=200
      >
      <template  slot-scope="scope">
        <span class='link_option' @click="linkToTrans(scope.row.pkHash)">{{$t('tableCont.transDetial')}}</span>
      </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @current-change="indexChange"
      :current-page="pageIndex"
      layout="total, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>
<script>
import API from '../API/resource.js'
export default {
  data () {
    return {
      loading: false,
      tableData: [],
      pageIndex: 1,
      total: 0
    }
  },
  methods: {
    indexChange (e) {
      this.pageIndex = e
      this.blockList()
    },
    onCopy () {
      this.$message({
        type: 'success',
        message: this.$t('tableCont.copySuccess')
      })
    },
    getLastData () {
      this.loading = true
      const url = '?groupId=' + this.groupId + '&pageNumber=' + this.pageIndex + '&pageSize=10&brokerId=' + this.brokerId
      API.blockList(url).then(res => {
        if (res.status === 200) {
          this.total = res.data.data.total
          const last = Math.ceil(res.data.data.total / 10)
          this.pageIndex = last
          const url = '?groupId=' + this.groupId + '&pageNumber=' + last + '&pageSize=10&brokerId=' + this.brokerId
          API.blockList(url).then(res => {
            if (res.status === 200) {
              this.tableData = res.data.data.pageData.reverse()
            }
          })
        }
      })
      this.loading = false
    },
    blockList () {
      this.loading = true
      const url = '?groupId=' + this.groupId + '&pageNumber=' + this.pageIndex + '&pageSize=10&brokerId=' + this.brokerId
      API.blockList(url).then(res => {
        if (res.status === 200) {
          this.tableData = res.data.data.pageData.reverse()
          this.total = res.data.data.total
        }
      })
      this.loading = false
    },
    linkToTrans (e) {
      sessionStorage.setItem('blockHash', e)
      this.$store.commit('set_active', '1-2')
      this.$store.commit('set_menu', [this.$t('sideBar.blockChainInfor'), this.$t('sideBar.transaction'), this.$t('sideBar.transactionDetial')])
      this.$router.push('./transactionInfor')
    }
  },
  mounted () {
    this.getLastData()
  },
  computed: {
    brokerId () {
      return this.$store.state.brokerId
    },
    groupId () {
      return this.$store.state.groupId
    }
  },
  watch: {
    groupId (nVal) {
      this.loading = true
      this.tableData = []
      if (nVal !== '-1') {
        this.getLastData()
      }
    }
  }
}
</script>
