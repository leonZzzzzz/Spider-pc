<template>
  <el-dialog
    :title="id ? '编辑小程序' : '新建小程序'"
    :visible.sync="visible"
    width="540px"
    :append-to-body="true"
    :before-close="close"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
  >
    <el-form :model="model" ref="model" label-width="100px" label-position="left">
      <el-form-item label="小程序分类" prop="groupId" :rules="[{ required: true, message: '该字段不能为空' }]">
        <el-cascader
          :options="data"
          :props="{ value: 'id', label: 'name', children: 'childs' }"
          :show-all-levels="true"
          v-model="model.groupId"
          style="width: 400px"
          @change="changeCascader"
          ref="elcascader"
          @visible-change="elCascaderOnlick"
          @expand-change="elCascaderOnlick"
          clearable
        ></el-cascader>
      </el-form-item>
      <el-form-item label="小程序标题" prop="name"  :rules="[{ required: true, message: '该字段不能为空' }]">
        <el-input placeholder="请输入小程序标题" v-model="model.name"></el-input>
      </el-form-item>
      <el-form-item label="appId" prop="programId"  :rules="[{ required: true, message: '该字段不能为空' }]">
        <el-input placeholder="请输入小程序appid" v-model="model.programId"></el-input>
      </el-form-item>
      <el-form-item label="路径" prop="link" :rules="[{ required: true, message: '该字段不能为空' }]">
        <el-input placeholder="请输入小程序page路径" v-model="model.link"></el-input>
      </el-form-item>
      <el-form-item label="小程序封面" prop="imageUrl"  :rules="[{ required: true, message: '该字段不能为空' }]">
        <QcImageUpload v-model="model.imageUrl" width="100px" height="100px"></QcImageUpload>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="close(false)">取 消</el-button>
      <el-button type="primary" @click="success()" :loading="loading">确 定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  props: {
    visible: Boolean,
    id: {
      type: String,
      default: ''
    },
  },
  data() {
    return {
      model: {
        name:'',
        groupId:'',
        link:'',
        imageUrl:'',
        programId:'',
        type:'program',
      },
      data:[],
      loading:false
    }
  },

  watch: {
    visible(val) {
      this.getDepartmentTreeData()
      if (val) {
        if (this.id) {
          this.getCarousel()
        }
      }
    }
  },

  methods: {
    // 获取分类数据
    getDepartmentTreeData() {
      this.$http.getMateTree({ type:'program' }).then((res) => {
        this.data = res.data.data
        this.data.forEach((item) => {
          if (item.childs.length == 0) {
            item.childs = null
          }
        })
      })
    },
    // 获取分类id
    changeCascader(e){
      if(e.length==1){
        this.model.groupId=e[0]
      }
      if(e.length==2){
        this.model.groupId=e[1]
      }
    },
    
    getCarousel() {
      this.$http.materialDetail({ id: this.id }).then(res => {
        this.model = res.data.data
      })
    },
    success() {
      if(this.model.type==''){
        this.$message.error('请选择文本分类')
        return
      }
      if(this.model.content==""){
        this.$message.error('请填写文本内容')
        return
      }
      this.$refs.model.validate(valid => {
        if (valid) {
          let model = JSON.parse(JSON.stringify(this.model))
          this.loading = true
          this.saveModel(this.id ? 'materialUpdate' : 'materialAdd', model)
        }
      })
    },
    saveModel(type, model) {
      this.$http[type](model)
        .then(() => {
          this.loading = false
          this.$message({
            message: type === 'materialUpdate' ? '修改成功' : '添加成功',
            type: 'success'
          })
          this.close(true)
        })
        .catch(() => {
          this.loading = false
        })
    },
    close(flag) {
      this.model = this.$options.data().model
      this.$nextTick(() => {
        this.$refs.model.clearValidate()
      })
      this.$emit('closeMini', flag)
    },
    elCascaderOnlick(e){
      let that = this;
      setTimeout(function() {
        document.querySelectorAll(".el-cascader-node__label").forEach(el => {
          el.onclick = function() {
            // this.previousElementSibling.onclick();
            // that.$refs.elcascader.dropDownVisible = false;
          };
        });
        document.querySelectorAll(".el-cascader-panel .el-radio").forEach(el => {
          el.onclick = function() {
              // that.$refs.elcascader.dropDownVisible = false;
            };
          });
      }, 100);
    },
  }
}
</script>
<style>
</style>
