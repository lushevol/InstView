<template>
  <div class="tags">
    <el-card>
      <el-form :inline="true" v-model="inputValue" size="mini">
        <div class="title">
          <div v-for="key in tagKeys" :key="`${key}_title}`" v-text="tagProp[key]['label']"></div>
        </div>
        <div class="tags-select" v-for="(item, index) in inputValue" :key="index">
          <!-- 单个tag的内容 -->
          <el-form-item v-for="key in tagKeys" :key="`${key}_${index}`">
            <TypesInput
              widthClass="short"
              :extra="{ 'tag': { key, index }}"
              :itemProp="tagProp[key]"
              @update:dataProp="updateData"
              :dataProp="item[key]">
            </TypesInput>
          </el-form-item>
          <el-button class="tag-minus" v-if="(index +1) !== inputValue.length" @click="inputValue.splice(index, 1)" icon="el-icon-minus" size="mini" type="danger" circle></el-button>
          <el-button class="tag-plus" v-else @click="addTag()" icon="el-icon-plus" size="mini" type="primary" circle></el-button>
        </div>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import TypesInput from './typesInput'

export default {
  name: 'TagsInput',
  data() {
    return {
      inputValue: []
    }
  },
  props: {
    dataProp: { //初始化数据
      type: Array,
      required: true
    },
    itemProp: { // item全部字段信息，也包含tags
      type: Object,
      required: true
    }
  },
  watch: {
    inputValue: {
      handler(val) {
        this.$emit('update:dataProp', { prop: this.itemProp.prop, value: val })
      },
      deep: true
    }
  },
  computed: {
    /**
     * @description: tags的内容
     * @param {type} 
     * @return: 
     */
    tagProp() {
      return this.itemProp.tags
    },
    tagKeys() {
      return Object.keys(this.tagProp)
    },
    /**
     * @description: 初始化行
     * @param {type} 
     * @return: 
     */
    tagEmptyLine() {
      let obj = {}
      this.tagKeys.map(item => {
        obj[item] = ''
      })
      return obj
    }
  },
  methods: {
    // 增加一个空序列
    addTag() {
      const emptyLine = JSON.parse(JSON.stringify(this.tagEmptyLine))
      this.inputValue.push(emptyLine)
    },
    /**
     * @description: 底层触发的更新数据
     * @param {type} 
     * @return: 
     */
    updateData({ prop, value, extra }) {
      const key = extra['tag']['key']
      const index = extra['tag']['index']
      this.inputValue[index][key] = value
    },
    /**
     * @description: 初始化数据
     * @param {type} 
     * @return: 
     */
    initComponent() {
      this.inputValue = JSON.parse(JSON.stringify(this.dataProp))
    }
  },
  components: {
    TypesInput
  },
  mounted() {
    this.initComponent()
  }
}
</script>

<style lang="scss" scope>
  .tags {
    margin-top: 10px;
    .el-card__body {
      padding-top: 10px;
    }
    .title {
      display: flex;
      div {
        margin: 0 auto;
      }
    }
    .tags-select {
      .el-form-item--mini.el-form-item {
        margin-bottom: 0;
      }
      .tag-plus, .tag-minus {
        position: relative;
        bottom: 7px;
      }
    }
  }
</style>