<template>
  <div class="table-box" :style="{ height: `${tHeight}px` }">
    <el-table
      v-loading="loading"
      :data="tableData"
      @selection-change="handleSelectionChange"
      :highlight-current-row="currentRow"
      @row-click="handleRowClick"
      :height="tHeight - 52"
      style="overflow-y: auto;"
    >
      <el-table-column type="selection" width="55" align="center" v-if="!currentRow"></el-table-column>
      <!-- <template v-for="(item, i) in propData" :key="i"> -->
      <el-table-column
        v-if="!item.template"
        v-for="(item, i) in propData"
        :key="i"
        :prop="item.prop"
        :label="item.label"
        :align="item.align || 'center'"
      ></el-table-column>
      <!-- </template> -->
      <!--布尔值  -->
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'Boolean'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] ? '是' : '否' }}</template>
      </el-table-column>
      <!--图片  -->
      <el-table-column align="center" v-else-if="item.template == 'img'" :label="item.label" :width="140">
        <template slot-scope="scope">
          <img
            style="display:block;margin:10px auto"
            src="http://athena-1255600302.cosgz.myqcloud.com/attachments/qc-logo.png"
            width="60"
            height="60"
            v-if="scope.row[item.prop] == undefined || scope.row[item.prop] === ''"
          />
          <img
            style="display:block;margin:10px auto"
            :src="`${imgHost}${scope.row[item.prop]}`"
            width="60"
            height="60"
            v-else-if="item.isPrefix"
          />
          <img
            style="display:block;margin:10px auto"
            :src="`${scope.row[item.prop]}`"
            width="60"
            height="60"
            v-else-if="item.headImage"
          />
          <img style="display:block;margin:10px auto" :src="scope.row[item.prop]" width="60" height="60" v-else />
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      style="text-align: center; padding: 10px; 0"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="1"
      :page-sizes="[10, 20, 30]"
      :page-size="page.pageSize"
      layout="total, sizes, prev, pager, next"
      :total="page.total"
    ></el-pagination>
  </div>
</template>

<script>
export default {
  props: {
    tableData: Array,
    propData: Array,
    activeIndex: String,
    page: Object,
    currentRow: {
      default: false,
      type: Boolean
    },
    loading: Boolean
  },
  data() {
    return {
      tHeight: 0
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.getHeightTable()
      window.onresize = () => {
        this.getHeightTable()
      }
    })
  },
  methods: {
    getHeightTable() {
      let search = document.querySelector('.search-wrap')
      let button = document.querySelector('.button-wrap')
      this.tHeight = window.innerHeight - 100 - 2
      if (search) this.tHeight = this.tHeight - search.offsetHeight
      if (button) this.tHeight = this.tHeight - button.offsetHeight
    },
    handleSizeChange(val) {
      this.$emit('size-change', val)
    },
    handleCurrentChange(val) {
      this.$emit('current-change', val)
    },
    handleSelectionChange(val) {
      this.$emit('get-selection', val)
    },
    handleRowClick(row) {
      this.$emit('row-click', row)
    }
  },
  filters: {
    // 状态
    signStatus(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '待审核'
        case 2:
          return '审核通过'
        case 3:
          return '审核不通过'
        case 4:
          return '待支付'
        case 5:
          return '已支付'
        case 6:
          return '已报名'
        case 7:
          return '已退款'
        case 8:
          return '已取消'
      }
    },
    // 签到方式
    checkWay(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '手机签到'
        case 2:
          return '暗号签到'
        case 3:
          return '二维码'
        case 5:
          return '空降嘉宾'
      }
    },
    // 评论状态
    auditStatus(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '未审核'
        case 2:
          return '已审核'
        case 3:
          return '已拒绝'
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.table-box {
  border: 1px solid #f3f3f3;
  height: 600px;
  .el-table thead tr th {
    background-color: #f5f7fa;
  }
}
</style>
