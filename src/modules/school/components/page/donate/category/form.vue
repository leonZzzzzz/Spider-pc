<template>
  <el-dialog :title='id ? "修改" : "添加"' :visible.sync="visible" width="880px" :append-to-body="true" :before-close="close" :close-on-click-modal="false" :close-on-press-escape="false">
    <div style="overflow: auto;" v-loading="dataLoading">
      <el-form :model="model" ref="model" class="new-form" label-position="top">
        <el-form-item class="item-text">
          <el-form-item label="名称" prop="name" :rules="[{ required: true, message: '该字段不能为空'}]">
            <el-input size="small" v-model="model.name" clearable></el-input>
          </el-form-item>
          <el-form-item label="备注">
            <el-input size="small" v-model="model.info" clearable></el-input>
          </el-form-item>
          <el-form-item label="排序">
            <el-input-number size="small" v-model="model.seqNum"></el-input-number>
          </el-form-item>
        </el-form-item>
      </el-form>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button size="small" @click="close(false)">取 消</el-button>
      <el-button size="small" type="primary" @click="success()" :loading="loading">确 定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import api from "apiSchool/common";

export default {
  props: {
    visible: Boolean,
    id: {
      type: String,
      default: ""
    },
  },
  data() {
    return {
      model: {
        name: "",
        type: 5,
        seqNum: 0,
        info: ""
      },
      loading: false,
      dataLoading: false,
    };
  },
  watch: {
    visible(val) {
      if (val) {
        if (this.id != "") {
          this.dataLoading = true;
          this.detail();
        }
      }
    }
  },
  methods: {
    //获取
    detail() {
      api.getCategory({ id: this.id }).then(res => {
        this.model = res.data.data;
        this.dataLoading = false;
      });
    },
    //添加
    save() {
      api
        .addCategory(this.model)
        .then(res => {
          this.loading = false;
          this.$message({
            message: "添加成功",
            type: "success"
          });
          this.close(true);
        })
        .catch(err => {
          this.loading = false;
        });
    },
    //修改
    update() {
      api
        .updateCategoryJson(this.model)
        .then(res => {
          this.loading = false;
          this.$message({
            message: "修改成功",
            type: "success"
          });
          this.close(true);
        })
        .catch(err => {
          this.loading = false;
        });
    },
    success() {
      this.$refs.model.validate(valid => {
        if (valid) {
          this.loading = true;
          if (this.id != "") {
            this.update();
          } else {
            this.save();
          }
        } else {
          return false;
        }
      });
    },
    close(flag) {
      this.model = this.reduc();
      this.$emit("close", flag);
    },
    reduc() {
      let model = {
        name: "",
        type: 5,
        seqNum: 0,
        info: ""
      }
      return model
    },
  }
};
</script>

