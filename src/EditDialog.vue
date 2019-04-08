<template>
  <el-dialog :visible.sync="show.dialogVisible" class="edit-dialog" width="80vw" top="5vh">
    <div slot="title">
      <slot name="title"></slot>
    </div>
    <InnerForm
      ref="inner-form"
      @paramChange = "updateParam"
      :referTable="referTable"
      :selectOptions="selectOptions"
      :dataProps="dataProps">
    </InnerForm>
    <div slot="footer" class="dialog-footer">
      <el-button @click="closeDialog">取 消</el-button>
      <el-button type="primary" @click="Submit">
        <slot name="submit-text">
          保 存
        </slot>
      </el-button>
      <slot name="extra-buttons"></slot>
    </div>
  </el-dialog>
</template>

<script>
import InnerForm from './InnerForm'

export default {
  name: 'EditDialog',
  props: {
    dataProps: {
      type: Object,
      required: true
    },
    referTable: {
      type: Array,
      required: true
    },
    selectOptions: {
      type: Object
    }
  },
  watch: {
    'show.dialogVisible': {
      handler(newVal) {
        // 窗口打开时初始化数据
        if(newVal) {
          this.dataInitial()
        } else {
          this.clearData()
        }
      }
    }
  },
  data() {
    return {
      show: {
        dialogVisible: false
      }
    }
  },
  methods: {
    /**
     * @description: 修改某个字段的时候通知父组件
     * @param { prop, value} key & value
     * @return:
     */
    updateParam({ prop, value}) {
      this.$emit('paramChange', { key: prop, value})
    },
    Submit() {
      this.$refs['inner-form'].Submit().then((result) => {
        const { data, editedProp } = result  
        if(!data) return false
        this.$emit('submit', { data, editedProp })
        // setTimeout(() => {this.$emit('refresh')}, 1000)
        this.closeDialog()
      }).catch((err) => {
        
      }).finally(() => {
      });
    },
    dataInitial() {
      if(this.$refs['inner-form']) {
        setTimeout(() => {
          this.$refs['inner-form'].dataInitial()  // 防止比tab切换数据更早触发
        }, 0)
      } else {
        setTimeout(() => {
          this.initPageData()
        }, 100)
      }
    },
    formDataOnChange({ editedProp }) {
      this.$emit('formDataOnChange', { editedProp })
    },
    clearData() {
      this.$refs['inner-form'].clearData()
    },
    closeDialog() {
      this.show.dialogVisible = false
    },
    initPageData() {
      this.dataInitial()
    },
    initPage() {
      this.initPageData()
    }
  },
  components: {
    InnerForm
  }
}
</script>

<style lang="scss">
  .edit-dialog {
    .el-dialog {
      .el-dialog__body {
        padding: 0;
      }
    }
  }
</style>
