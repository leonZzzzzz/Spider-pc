<template>
  <div class="qc-table" :style="{ height: `${autoHeight || tHeight}px` }">
    <!-- @row-click="handleSelect" @selection-change="handleCheck"  -->
    <el-table
      v-loading="loading"
      element-loading-text="数据加载中"
      height="100%"
      style="overflow-y: auto;"
      :data="tableData"
      highlight-current-row
      @row-click="vuexTableList"
      @cell-dblclick="DbTableList"
      :stripe="isStripe"
      @selection-change="vuexTableId"
    >
      <!-- @cell-dblclick='DbTableList'  -->
      <!-- <el-table-column
        type="selection"
        width="55">
      </el-table-column> -->
      <!--基本的显示  -->
      <el-table-column
        v-for="(item, index) in tableList"
        :key="index"
        :align="item.align || 'center'"
        v-if="!item.template"
        :prop="item.prop"
        :label="item.label"
        :width="item.width"
        :show-overflow-tooltip="!overflow"
      >
      </el-table-column>
      <!--序号  -->
      <el-table-column align="center" v-else-if="item.template == 'index'" type="index" label="序号" width="65">
      </el-table-column>
      <!--排序  -->
      <el-table-column align="center" v-else-if="item.template == 'seqNum'" :label="item.label" width="65">
        <template slot-scope="scope">{{ scope.row[item.prop] }}</template>
      </el-table-column>
      <!--checkbox  -->
      <el-table-column
        :align="item.align || 'center'"
        type="selection"
        v-else-if="item.template == 'checkbox'"
        width="55"
      >
      </el-table-column>
      <!--金钱  -->
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'price'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] || 0 | price }}</template>
      </el-table-column>
      <!--布尔值  -->
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'Boolean'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] ? '是' : '否' }}</template>
      </el-table-column>
      <!--性别  -->
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'sex'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] == 1 ? '男' : '女' }}</template>
      </el-table-column>
      <!--是否启用  -->
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'used'"
        :label="item.label"
        :width="item.width"
        inline-template
      >
        <div>{{ row[item.prop] == 1 ? '是' : '否' }}</div>
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
            :src="`${imgHost}${scope.row[item.prop]}`"
            width="100"
            height="100"
            v-else-if="item.isQR"
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
      <!--按钮  -->
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'btn'"
        :label="item.label"
        :width="item.width"
        inline-template
      >
        <div>
          <el-button @click="handleAdd(row)" size="small">{{ item.prop }}</el-button>
        </div>
      </el-table-column>
      <!--日期 -->
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'date'"
        :label="item.label"
        :width="item.width"
        inline-template
      >
        <div>
          {{ row[item.prop] | formatData }}
        </div>
      </el-table-column>
      <!-- 时间范围 -->
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'timeLimit'"
        :label="item.label"
        :width="item.width"
        inline-template
      >
        <div>
          {{ row.startTime | yearMoutn }}&nbsp;&nbsp;&nbsp;&nbsp;{{ row.startTime | DateTime }}-{{
            row.endTime | DateTime
          }}
        </div>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'array'"
        :label="item.label"
        :width="item.width"
        inline-template
      >
        <div>
          <span v-for="(arrayList, index) in row[item.prop]" :key="index"
            >{{ arrayList }}
            <span v-if="index + 1 != row[item.prop].length">；</span>
          </span>
        </div>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'tagArray'"
        :label="item.label"
        :width="item.width"
        inline-template
      >
        <div>
          <span v-for="(tag, index) in row[item.prop]" :key="index"
            >{{ tag.name }}<span v-if="index + 1 != row[item.prop].length">/</span>
          </span>
        </div>
      </el-table-column>

      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'audit'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | audit }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'auditStatus'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | auditStatus }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'auditRegisterStatus'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | auditRegisterStatus }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'payWay'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | payWay }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'couponStatus'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | couponStatus }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'payStatus'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | payStatus }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'merchant'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | merchant }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'tagType'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | tagType }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'ruleType'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | ruleType }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'disAudit'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | disAudit }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'checkWay'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | checkWay }}</template>
      </el-table-column>
      <el-table-column
        :align="item.align || 'center'"
        v-else-if="item.template == 'professionType'"
        :label="item.label"
        :width="item.width"
      >
        <template slot-scope="scope">{{ scope.row[item.prop] | professionType }}</template>
      </el-table-column>
    </el-table>
    <div class="qc-block">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="page.pageNum"
        :page-sizes="[10, 20, 30]"
        :page-size="page.pageSize"
        layout="total, sizes, prev, pager, next"
        :total="page.total"
      >
      </el-pagination>
    </div>
    <!-- <dig-qrcode :visible="digQrcode" :imgUrl="row[item.prop]" @close="checkQrcode"></dig-qrcode> -->
  </div>
</template>

<script>
import apiCommon from '@/api'
import { mapMutations } from 'vuex'
import qs from 'qs'
export default {
  props: {
    overflow: Boolean,
    tableList: Array,
    search: Object,
    url: String,
    delay: Boolean,
    height: Number,
    isDetail: Boolean,
    isStripe: Boolean,
    tongji: {
      type: Number,
      default: 0
    },
    autoHeight: Number,
    categoryType: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      imgURL: '',
      loading: false,
      tableData: [],
      page: {
        total: 0,
        pageSize: 20,
        pageNum: 1
      },
      digQrcode: false,
      tableRow: {},
      tHeight: 0
    }
  },
  created() {
    this.imgURL = process.env.IMG_URL
  },
  mounted() {
    this.$nextTick(() => {
      this.getHeightTable()
      window.onresize = () => {
        this.getHeightTable()
      }
      this.getTableList()
    })
  },
  filters: {
    professionType(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '学位'
        case 2:
          return '系别'
        case 3:
          return '专业'
      }
    },
    // 业务员审核
    disAudit(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '待审核'
        case 2:
          return '已审核'
      }
    },
    // 订单审核状态
    audit(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '待审核'
        case 2:
          return '确认中'
        case 3:
          return '审核完毕'
        case 4:
          return '已拒绝'
      }
    },
    // 审核状态
    auditStatus(val) {
      if (val === '') return ''
      switch (val) {
        case 0:
          return '提交'
        case 1:
          return '审批通过'
        case 2:
          return '审批不通过'
      }
    },
    // 支付方式
    payWay(val) {
      if (val === '') return ''
      switch (val) {
        case 'alipay':
          return '支付宝支付'
        case 'cash':
          return '现金'
        case 'pos':
          return 'POS机'
        case 'wechat':
          return '微信支付'
      }
    },
    //支付状态
    payStatus(val) {
      if (val === '') return ''
      switch (val) {
        case -1:
          return '支付失败'
        case 1:
          return '新建'
        case 2:
          return '进行中'
        case 3:
          return '交易成功'
      }
    },
    //优惠券领取状态
    couponStatus(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '未用'
        case 2:
          return '已用'
        case 3:
          return '已过期'
      }
    },
    // 商户状态
    merchant(val) {
      if (val === '') return ''
      switch (val) {
        case 0:
          return '新建'
        case 1:
          return '开业'
        case 2:
          return '歇业'
        case 3:
          return '结业'
      }
    },
    // 标签类型
    tagType(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '会员'
        case 2:
          return '商品'
        case 3:
          return '活动'
        case 4:
          return '课程'
        case 5:
          return '新闻'
        case 6:
          return '产品'
      }
    },
    // 规则类型
    ruleType(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '商城'
        case 2:
          return '活动'
        case 3:
          return '课程'
      }
    },
    // 签到方式
    checkWay(val) {
      if (val === '') return ''
      switch (val) {
        case 1:
          return '二维码'
        case 2:
          return '手机签到'
        case 3:
          return '暗号签到'
      }
    }
  },
  methods: {
    getHeightTable() {
      let search = document.querySelector('.search-wrap')
      let button = document.querySelector('.button-wrap')
      let tongji = document.querySelector('.tongji')
      this.tHeight = window.innerHeight - 120 - 52
      if (search) this.tHeight = this.tHeight - search.offsetHeight
      if (button) this.tHeight = this.tHeight - button.offsetHeight
      if (tongji) this.tHeight = this.tHeight - tongji.offsetHeight
      if (this.isDetail) this.tHeight = this.tHeight - 50
    },
    // 表单请求接口地址
    apiTable(url, search) {
      this.loading = true
      var data = {}
      data = Object.assign(data, search)

      apiCommon
        .get(url, data)
        .then(res => {
          this.tableData = res.data.data.list
          console.log('数据', this.tableData)
          this.page.total = res.data.data.total
          this.loading = !true
        })
        .catch(err => {
          this.loading = !true
          console.log('表格接口调用失败', err)
          // this.$router.replace("/login");
        })
    },
    // 查询表格数据
    getTableList() {
      this.page = Object.assign(this.search, this.page)
      if (!this.categoryType) this.apiTable(this.url, this.page)
    },
    // 搜索表格数据
    searchTableList() {
      this.setTableList({})
      this.page = Object.assign(this.search, this.page)
      this.page.pageNum = 1
      this.apiTable(this.url, this.page)
    },
    searchTableListKeep() {
      this.setTableList({})
      this.page = Object.assign(this.search, this.page)
      this.apiTable(this.url, this.page)
    },
    // 分页
    handleSizeChange(val) {
      document.getElementsByClassName('el-table__body-wrapper')[0].scrollTop = 0
      this.page.pageSize = val
      this.page.pageNum = 1
      this.getTableList()
    },
    handleCurrentChange(val) {
      document.getElementsByClassName('el-table__body-wrapper')[0].scrollTop = 0
      this.page.pageNum = val
      this.getTableList()
    },
    // 每行选中 提交到vuex 管理
    vuexTableList(row) {
      this.setTableList(row)
    },
    //双击行
    DbTableList(row) {
      this.$emit('DbTableList', row)
    },
    // 每行选中id 提交到vuex 管理
    vuexTableId(row) {
      let arr = ''
      row.forEach(item => {
        arr += '_' + item.id
      })
      this.setTableId(arr.substr(1))
    },
    // vuex
    ...mapMutations({
      setTableList: 'tableOld/TABLELIST',
      setTableId: 'tableOld/TABLEID'
    })
  },
  destroyed() {
    this.setTableList({})
  }
}
</script>

<style lang="scss" scoped>
// @import url("../../assets/styles/wawa.scss");
// @import "@/assets/styles/element-variables.scss";
.img {
  width: 200px;
  img {
    width: 100%;
  }
}
// .qc_table_success {
//   color: #35cd02;
//   vertical-align: middle;
//   &::before {
//     content: "";
//     .dot(#35cd02,5px,5px);
//   }
// }
// .qc_table_error {
//   color: #fd0a01;
//   vertical-align: middle;
//   &::before {
//     content: "";
//     .dot(#fd0a01,5px,5px);
//   }
// }
.el-table__body tr.current-row > td {
  // background: #50bfff !important;
  background: $--color-primary !important;
  color: #fff !important;
}
.qc-block {
  margin-bottom: 0;
  padding: 10px 0;
  text-align: center;
  // margin-bottom: 50px;
}
</style>
