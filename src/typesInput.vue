<!--
 * @Description: 各种类型的输入框，包括input、select、tags、dateinput等
 * @Author: lushevol
 * @LastEditors: lushevol
 * @Date: 2019-03-29 16:40:22
 * @LastEditTime: 2019-04-12 16:05:32
 -->
<template>
  <div class="types-input">
    <!-- 多选框 -->
    <el-select 
      v-if="componentType === 'select'" 
      v-model="inputValue" 
      :placeholder="item.placeholder || '请选择'"
      :class="[widthClass]"
      size="mini" 
      clearable
      :disabled="isEditable(item.editable)">
      <el-option
        v-for="(opt, optIndex) in item.options"
        :key="optIndex"
        :value="opt.value || opt"
        :label="opt.note || opt.label || opt">
      </el-option>
    </el-select>
    <!-- textarea -->
    <el-input
      v-else-if="componentType === 'textarea'"
      v-model="inputValue"
      :class="[widthClass]"
      :placeholder="item.placeholder"
      :type="item.type"
      :rows="item.rows"
      :autosize="{ minRows: 2, maxRows: 4}"
      size="mini"
      :disabled="item.editable === false">
    </el-input>
    <!-- 日期选择框 -->
    <el-date-picker
      v-else-if="componentType === 'date'"
      size="mini"
      v-model="inputValue"
      :class="[widthClass]"
      :type="item.type"
      :format="item.format"
      :disabled="isEditable(item.editable)"
      clearable
      range-separator="-"
      start-placeholder="开始日期"
      end-placeholder="结束日期">
    </el-date-picker>
    <!-- 文本输入框 -->
    <el-input 
      v-else
      v-model="inputValue" 
      :class="[widthClass]"
      :placeholder="item.placeholder"
      autocomplete="off"
      size="mini" 
      clearable
      :disabled="isEditable(item.editable)">
    </el-input>
  </div>
</template>

<script>
export default {
  name: 'TypesInput',
  data() {
    return {
      inputValue: ''
    }
  },
  watch: {
    inputValue(val) {
      this.$emit('update:dataProp', { prop: this.item.prop, value: val, extra: this.extra, alias: this.item.submitProp })
    },
    dataProp: {
      handler(val) {
        this.initComponent()
      },
      immediate: true,
      deep: true
    }
  },
  props: {
    dataProp: {
      required: true
    },
    itemProp: {
      type: Object,
      required: true
    },
    extra: {
      type: Object
    },
    widthClass: {
      type: String,
      default: () => {
        return ''
      }
    }
  },
  computed: {
    item() {
      return this.itemProp
    },
    componentType() {
      let compType = 'text'  // 组件类型：text 普通输入框，select 下拉框，date：日期选择
      if(this.item.options) {
        compType = 'select'
      } else if(['year', 'month', 'date', 'dates', 'week', 'datetime', 'datetimerange', 'daterange', 'monthrange', 'time'].includes(this.item.type)) {
        compType = 'date'
      } else if (this.item.type){
        compType = this.item.type
      }
      return compType
    }
  },
  methods: {
    initComponent() {
      if (Object.prototype.toString.call(this.dataProp) === '[object Array]'){
        // TODO: 因为服务器mac字段是 Array，而不涉及到编辑，因此转成 JSON String 展示
        this.inputValue = JSON.stringify(this.dataProp)
      } else {
        this.inputValue = JSON.parse(JSON.stringify(this.dataProp))
      }
    },
    isEditable(itemEditable) {
      // 如果没有设置editable，则默认可编辑
      if(itemEditable === undefined) {
        return false
      } else if(typeof itemEditable === 'boolean') {
        return !itemEditable
      } else if(typeof itemEditable === 'function') {
        return !itemEditable(this)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .types-input {
    .el-input, .el-select, .el-textarea {
      width: 300px;
      &.short {
        width: 200px;
      }
    }
  }
</style>
