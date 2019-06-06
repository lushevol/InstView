<template>
  <IndexPage class="example">
    <template slot="title-text">
      <slot name="title-text"></slot>
    </template>
    <template slot="operation">
      <!-- create 按钮 -->
      <el-button v-if="isAllowed2Add" size="mini" @click="createNode" type="primary">
        <slot name="create-btn-text">增加</slot>
      </el-button>
      <!-- 额外operation slot -->
      <slot name="header-operation"></slot>
    </template>
    <div slot="content">
      <!-- table -->
      <InstantTable
        @edit="editNode"
        @delete="deleteNode"
        :optionsProps="tableOptions"
        :tableHeaderProps="table.header"
        :tableDataProps="tableData"
        :methodFunctions="methodFunctions">
        <template slot="operation-extra-btns" slot-scope="scope">
          <slot name="table-operation-extra-btns" :scope="scope"></slot>
        </template>
      </InstantTable>
      <!-- 新增和编辑的弹窗 -->
      <EditDialog
        ref="editDialog"
        :configs="configs"
        @formDataOnChange="dialogFormDataOnChange"
        @submit="updateData"
        :selectOptions="selectOptions"
        :referTable="table.header"
        :dataProps="editData">
        <div slot="title">
          {{dialogTitle}}
        </div>
      </EditDialog>
    </div>
  </IndexPage>
</template>

<script>
// components
import IndexPage from '@/components/Page/IndexLayout'
import InstantTable from './InstantTable'
import EditDialog from './EditDialog'

// mixins
// import CmdbSelectOptions from '@/views/CMDB/mixins/CMDBSelectOptions'

export default {
  data() {
    return {
      tableData: [],
      editData: {},
      tableOptions: {
        defaultLoading: true
      },
      type: '' // 弹窗作用，编辑update/新增create
    }
  },
  props: {
    methodFunctions: {
      type: Object,
      required: true
    },
    table: {
      type: Object,
      required: true
    },
    configs: {
      type: Object
    },
    selectOptions: {
      type: Object
    },
    isAllowed2Add: {
      type: Boolean,
      default: true
    }
  },
  mixins: [
    // CmdbSelectOptions
  ],
  methods: {
    // 提交增加(create)/编辑(update) 数据
    updateData({ originData, data, editedProp }) {
      this.methodFunctions[this.type]({ originData, data, editedProp }).then((result) => {
        this.$message.success(`${this.dialogTitle} 成功!`)
        this.queryData()
      }).catch((err) => {
        this.$message.error(`${this.dialogTitle} 失败!`)
        console.error(err)
      });
    },
    // 提交删除(delete) 数据
    deleteData({ data, index }) {
      this.methodFunctions.delete({ data }).then((result) => {
        this.tableData.splice(index, 1)
        this.$message.success('删除成功!')
        // this.initPageData() // 删除完数据不需要调用 initPageData
      }).catch((err) => {
        this.$message.error('删除失败!')
        console.error(err)
      });
    },
    // 查询(read)数据
    queryData(data) {
      this.methodFunctions.read({ data }).then((result) => {
        if(result) {
          this.tableData = result
        }
      }).catch((err) => {
        this.$message.error('表格初始化失败!')
        console.error(err)
      });
    },
    createNode() {
      this.editData = {}
      this.type = 'create'
      this.$emit('createNode')
      this.showDialog('editDialog')
    },
    editNode({ rowData }) {
      this.editData = rowData
      this.type = 'update'
      this.$emit('editNode')
      this.showDialog('editDialog')
    },
    deleteNode({ rowData, index }) {
      this.deleteData({ data: rowData, index })
    },
    /**
     * @description: dialog中的form数据变更触发
     * @param {type} 
     * @return: 
     */
    dialogFormDataOnChange({ editedProp }) {
      this.$emit('dialogFormDataOnChange', { editedProp })
    },
    showDialog(type = 'editDialog') {
      this.$refs[type].show.dialogVisible = true // 显示弹窗
    },
    initPage() {
      this.initPageData()
    },
    initPageData() {
      // this.getSelectOptions()
      this.queryData()
    }
  },
  computed: {
    dialogTitle() {
      let title = ''
      switch (this.type) {
        case 'update':
          title = '编辑'
          break;
        case 'create':
          title = '新增'
          break;
        default:
          break;
      }
      return title
    }
  },
  mounted() {
    this.initPage()
  },
  components: {
    IndexPage,
    InstantTable,
    EditDialog
  }
}
</script>

<style lang="scss">

</style>
