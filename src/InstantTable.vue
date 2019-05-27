<template>
  <div class="crud-table">
    <search-table
      @onChecklistChange="onChecklistChange"
      @selectRows="selectRows"
      :tableDataProps="tableData" 
      :tableHeaderProps="tableHeader" 
      :optionsProps="tableOptions"
      ref="table">
      <div :slot="item.prop" slot-scope="scope" v-for="(item, index) in tableHeader" :key="index" v-if="item.prop !== 'operation'">
        <el-tag v-if="item.el === 'tag' && getFieldValue(scope.data.row, item).text" v-text="getFieldValue(scope.data.row, item).text" :type="getFieldValue(scope.data.row, item).type"></el-tag>
        <span v-else v-text="getFieldValue(scope.data.row, item).text"></span>
      </div>
      <template slot="operation" slot-scope="scope">
        <el-button size="mini" v-if="methodFunctions.showEdit ? methodFunctions.showEdit(scope.data.row) : true" type="primary"
                   @click="editAction(scope.data)" :disabled="methodFunctions.editDisabled ? methodFunctions.editDisabled(scope.data.row) : false">{{ getButtonText('edit') }}</el-button>
        <el-button size="mini" v-if="methodFunctions.showDelete ? methodFunctions.showDelete(scope.data.row) : true" type="danger"
                   @click="deleteAction(scope.data)">{{ getButtonText('delete') }}</el-button>
        <slot name="operation-extra-btns"></slot>
      </template>
    </search-table>
  </div>
</template>

<script>
import { searchTable } from '@/components/Table'

export default {
  name: 'InstantTable',
  data() {
    return {
      options: {
        hasExpand: false
      },
      delPopoverVisible: []
    }
  },
  props: {
    tableDataProps: {
      type: Array,
      required: true
    },
    tableHeaderProps: {
      type: Array,
      required: true
    },
    optionsProps: {
      type: Object
    },
    methodFunctions: {
      type: Object,
      default:()=> {
        return {
          showEdit: ()=> {
            return true
          },
          showDelete: ()=> {
            return true
          },
          editDisabled: ()=> {
            return false
          }
        }
      }
    }
  },
  components: {
    searchTable
  },
  computed: {
    tableData() {
      return this.tableDataProps
    },
    tableHeader() {
      return this.tableHeaderProps
    },
    tableOptions() {
      return this.optionsProps
    }
  },
  methods: {
    /**
     * @description: 显示的值的对照表
     * @param {type} {Array} 值和显示文字的对照表[{label: 'value', value: 'key'}]，
     *               {Function({ rawVal })} return 显示的文字, 
     *               {Object} key-value 对照
     * @return: 
     */
    getFieldValue(rowData, item) {
      const rawVal = rowData[item.prop]
      if(item.refer) {
        const referType = typeof item.refer
        if(referType === 'object') {
          const objectType = Object.prototype.toString.call(item.refer)
          if(objectType === '[object Array]') {
            let text = rawVal
            let type = ''
            item.refer.map(itemRow => {
              if(itemRow.value === text) {
                text = itemRow.label
                type = itemRow.type
              }
            })
            return { text, type }
          } else if(objectType === '[object Object]') {
            if(typeof item.refer[rawVal] === 'object') {
              return { text: item.refer[rawVal].text || rawVal, type: item.refer[rawVal].type }
            } else {
              return { text: item.refer[rawVal] || rawVal }
            }
          }
        } else if (referType === 'function') {
          return { text: item.refer({ rawVal }) }
        }
      } else {
        return { text: rawVal }
      }
    },
    editAction(scopeData) {
      const currentPage = this.$refs['table'].tableOptions.pagination.currentPage
      const pageSize = this.$refs['table'].tableOptions.pagination.pageSize
      const index = scopeData.$index + (currentPage -1) * pageSize
      this.$emit('edit', { rowData: scopeData.row || {}, index })
    },
    deleteAction(scopeData) {
      this.$confirm('是否删除?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const currentPage = this.$refs['table'].tableOptions.pagination.currentPage
        const pageSize = this.$refs['table'].tableOptions.pagination.pageSize
        const index = scopeData.$index + (currentPage -1) * pageSize
        this.$emit('delete', { rowData: scopeData.row || {}, index })
      }).catch(() => {

      });
    },
    /**
     * @description: 上传选择的行数据
     * @param {type} 
     * @return: 
     */
    selectRows({ rowDatas }) {
      this.$emit('selectRows', { rowDatas })
    },
    /**
     * @description: 清空选择
     * @param {type} 
     * @return: 
     */
    clearSelection() {
      this.$refs['table'].clearSelection()
    },
    getButtonText(buttonType) {
      let buttonText = ''
      if (this.tableOptions && this.tableOptions.buttonText) {
        buttonText = this.tableOptions.buttonText[buttonType]
      }
      if (buttonText) {
        return buttonText
      }
      switch (buttonType) {
        case 'edit':
          buttonText = '编辑'
          break
        case 'delete':
          buttonText = '删除'
          break
      }
      return buttonText
    },
    onChecklistChange({ data }) {
      this.$emit('onChecklistChange', { data })
    }
  }
}
</script>
