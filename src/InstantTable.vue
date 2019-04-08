<template>
  <div class="crud-table">
    <search-table 
      @selectRows="selectRows"
      :tableDataProps="tableData" 
      :tableHeaderProps="tableHeader" 
      :optionsProps="tableOptions"
      ref="table">
      <div :slot="item.prop" slot-scope="scope" v-for="(item, index) in tableHeader" :key="index" v-if="item.prop !== 'operation'">
        <span v-text="getFieldValue(scope.data.row, item)"></span>
      </div>
      <template slot="operation" slot-scope="scope">
        <el-button size="mini" v-if="methodFunctions.showEdit ? methodFunctions.showEdit(scope.data.row) : true" type="primary"
                   @click="editAction(scope.data)">编辑</el-button>
        <el-button size="mini" v-if="methodFunctions.showDelete ? methodFunctions.showDelete(scope.data.row) : true" type="danger"
                   @click="deleteAction(scope.data)">删除</el-button>
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
          showDelete: ()=>{
            return true
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
            let res = rawVal
            item.refer.map(itemRow => {
              if(itemRow.value === res) {
                res = itemRow.label
              }
            })
            return res
          } else if(objectType === '[object Object]') {
            return item.refer[rawVal] || rawVal
          }
        } else if (referType === 'function') {
          return item.refer({ rawVal })
        }
      } else {
        return rawVal
      }
    },
    editAction(scopeData) {
      this.$emit('edit', { rowData: scopeData.row || {} })
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
    selectRows({ rowDatas }) {
      this.$emit('selectRows', { rowDatas })
    }
  }
}
</script>
