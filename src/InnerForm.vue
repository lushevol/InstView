<template>
  <el-form 
    class="inner-form" 
    :rules="rules" 
    :model="formData" 
    ref="form">
    <div v-for="(item, index) in formLines" :key="index">
      <el-form-item :label="item.label" :label-width="style.formLabelWidth" v-if="ifItemVisible({item, formData})" :prop="item.prop">
        <TagsInput 
          v-if="item.tags" 
          @update:dataProp="updateData"
          :dataProp="formData[item.prop]"
          :itemProp="item">
        </TagsInput>
        <TypesInput
          v-else
          :itemProp="item"
          @update:dataProp="updateData"
          :dataProp="formData[item.prop]">
        </TypesInput>
      </el-form-item>
    </div>
  </el-form>
</template>

<script>
import TypesInput from './typesInput'
import TagsInput from './tagsInput'

export default {
  name: 'InnerForm',
  data() {
    return {
      formLines: [],  // form的列
      formData: {},  // form的Key-Value数据
      originFormData: {}, // formData的初始值，用与跟新值比对
      editedProp: new Set(), // 编辑了的prop列表
      rules: {},
      style: {
        formLabelWidth: '130px'
      }
    }
  },
  props: {
    // form的初始值
    dataProps: {
      type: Object,
      required: true,
    },
    // form的对照表，也是form列的来源
    referTable: {
      type: Array,
      required: true
    },
    // form中出现的select的options，需要对应prop
    selectOptions: {
      type: Object
    }
  },
  watch: {
    /**
     * @description: 当选项有变更时，重新加载选项
     * @param {type} 
     * @return: 
     */
    selectOptions: {
      handler(newVal, oldVal) {
        this.optionsInitial()
      },
      deep: true
    },
    /**
     * @description: 当referTable变更时，重新渲染
     * @param {type} 
     * @return: 
     */
    referTable: {
      handler(newVal, oldVal) {
        if(newVal) {
          this.dataInitial()
        }
      },
      immediate: true, 
      deep: true
    }
  },
  methods: {
    // 触发提交
    Submit() {
      return new Promise((resolve, reject) => {
        // 如果有检测必填项
        if(Object.keys(this.rules).length > 0) {
          this.checkValid().then((result) => {
            const res = this.realSubmit()
            resolve(res)
          }).catch((err) => {
            reject(false)
          });
        } else {
          const res = this.realSubmit()
          resolve(res)
        }
      })
    },
    // 真正的提交
    realSubmit() {
      // 新建一个可编辑的对象
      let data = Object.assign({}, this.formData)
      // 遍历将prop替换成submitProp
      this.formLines.map(item => {
        if(item['submitProp']) {
          data[item.submitProp] = data[item.prop]
          delete data[item.prop]
        }
      })
      return { data, editedProp: this.editedProp }
    },
    /**
     * @description: 检测必填项是否都填了
     * @param {type} 
     * @return: 
     */
    checkValid() {
      return new Promise((resolve, reject) => {
        this.$refs['form'].validate((valid) => {
          if(valid) {
            resolve()
          } else {
            reject()
          }
        });
      })
    },
    /**
     * @description: 重置必填检测状态
     * @param {type} 
     * @return: 
     */
    resetValid() {
      this.$refs['form'].resetFields();
    },
    /**
     * @description: 更新formData指定字段
     * @param {type} 
     * @return: 
     */
    updateData({ prop, value }) {
      this.$set(this.formData, prop, value)
      this.editedProp.add(prop)
      this.$emit('paramChange', { prop, value })
    },
    /**
     * @description: 初始化form数据，以referTable为基准，从dataProps中获取初始数据
     * @param {type} 
     * @return: 
     */
    dataInitial() {
      this.clearData()
      this.referTable.map(refItem => {
        if(refItem.prop && refItem.prop !== 'operation') {
          const selectOptions = (this.selectOptions && (this.selectOptions[refItem.selectProp] || this.selectOptions[refItem.prop])) || false  // select的选项
          // 构建行
          this.formLines.push({
            label: refItem.label,             // 文本
            prop: refItem.prop,               // 属性名
            selectProp: refItem.selectProp,   // 下拉框属性名
            submitProp: refItem.submitProp,   // 提交时的属性名
            editable: refItem.editable,       // 是否可编辑
            tags: refItem.tags,               // {Object} 有值时，可增加。包含字段和数据类型
            refer: refItem.refer,             // 显示的值的对照表，{Array} 值和显示文字的对照表[{label: 'value', value: 'key'}]，{Function({ rawVal })} return 显示的文字, {Object} key-value 对照
            hidden: refItem.formHidden,       // 是否隐藏
            type: refItem.type || '',         // value的数据类型，类型如下：
                                              // [text] 文本,
                                              // [textarea]
                                              // [year/month/date/dates/week/datetime/datetimerange/daterange/monthrange] 日期
                                              // [time] 时间
            format: refItem.format,           // 规定value的格式，仅部分type生效
            rows: refItem.rows,               // textarea rows 属性
            options: selectOptions            // select的选项
          })

          // 赋值
          this.$set(this.formData, refItem.prop, this.dataProps[refItem.prop] || refItem.default || '') // 从传入数据中获取 || 从默认值中获取

          // 设置rules
          if(refItem.required) {
            this.$set(this.rules, refItem.prop, { required: true, message: `请${selectOptions ? '选择' : '填写'}${refItem.label}` })
          }
        }
      })
      this.originFormData = Object.assign({}, this.formData)
      // FIXME 防止rule检测出现错乱，如果不加，则会乱掉
      setTimeout(() => {
        this.resetFormRules()
        this.editedProp.clear()
      })
    },
    /**
     * @description: 遍历赋值options
     * @param {type} 
     * @return: 
     */
    optionsInitial() {
      this.formLines.map((item, index) => {
        this.formLines[index].options = (this.selectOptions && this.selectOptions[item.prop]) || false
      })
    },
    /**
     * @description: 清空数据
     * @param {type} 
     * @return: 
     */
    clearData() {
      this.formLines.splice(0)
      this.formData = {}
      this.originFormData = {}
      this.rules = {}
      this.editedProp.clear()
      // this.clearObjectData(this.formData)
      // this.clearObjectData(this.rules)
    },
    ifItemVisible({ item, formData }) {
      if(item.hidden) {
        if(typeof item.hidden === 'boolean') {
          return !item.hidden
        } else if (typeof item.hidden === 'function') {
          return item.hidden({ item, formData })
        } else {
          return true
        }
      } else {
        return true
      }
    },
    /**
     * @description: 清空一个对象的每项
     * @param {type} 
     * @return: 
     */
    clearObjectData(obj) {
      Object.keys(obj).map(item => {
        this.$set(obj, item, '')
      })
    },
    /**
     * @description: 重置formdata内容，暂时无用
     * @param {type} 
     * @return: 
     */
    resetFormData() { 
      this.clearObjectData(this.formData)
    },
    /**
     * @description: 重置rule参数
     * @param {type} 
     * @return: 
     */
    resetFormRules() {
      if(this.$refs['form']) {
        this.$refs['form'].resetFields();
      }
    }
  },
  components: {
    TypesInput,
    TagsInput
  },
  mounted() {
    // this.$nextTick(() => {
    //   this.editedProp.clear()
    // })
  },
}
</script>

<style lang="scss" scoped>
  .inner-form {
    .el-form-item {
      margin-bottom: 6px;
    }
  }
</style>
