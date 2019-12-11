<template>
  <div class="container">
    <el-form ref="dataForm" :model="formData">
      <el-table :data="formData.tableData">
        <el-table-column>
          <template slot="header" slot-scope="scope">
            <el-checkbox :indeterminate="isIndeterminate" v-model="isAllChoose" @change="handleCheckAllChange(scope)" />
          </template>
          <template slot-scope="scope">
            <el-checkbox :ref="`checkbox${scope.$index}`" @change="handleClick(scope)" v-model="checkbox[scope.$index]" />
          </template>
        </el-table-column>
        <el-table-column
          prop="name"
          label="名字"
        />
        <el-table-column
          label="描述"
        >
          <template slot-scope="scope">
            <el-form-item
              :prop="`tableData.${scope.$index}.description`"
              :rules="
              { required: true, message: '来点信息', trigger: 'blur' }
            "
            >
              <el-input v-model="scope.row.description" />
            </el-form-item>
          </template>
        </el-table-column>
      </el-table>
      <el-button @click="btnClick">
        提交
      </el-button>
    </el-form>
  </div>
</template>

<script>

export default {
  data () {
    return {
      isAllChoose: false, // 是否全选
      isIndeterminate: false, // 有选择项但是没有全选
      checkbox: [], // checkbox绑定的数组 长度同tableData
      checkList: [], // 已选择的checkbox的值
      formData: {
        tableData: [ // table数据
          { id: '1', name: 'fengjialue', description: 'a handsome boy, yo~~' },
          { id: '2', name: 'suibianle', description: '' }
        ]
      }
    }
  },
  computed: {},
  created () {
    // 初始化checkbox
    this.checkbox.length = this.formData.tableData.length
    this.checkbox.fill(false)
  },
  methods: {
    // 全选
    handleCheckAllChange () {
      const temp = []
      temp.length = this.checkbox.length
      temp.fill(this.isAllChoose)
      this.$set(this, 'checkbox', temp) // $set能更新dom 不使用能正常赋值但页面不会有checkbox被选中的样式
      this.isIndeterminate = false
      this.checkList = this.isAllChoose ? this.formData.tableData : []
    },
    handleClick (scope) {
      // 判断被选中的个数
      const chooseNumber = this.checkbox.reduce((a, b) => {
        return a + b
      })

      this.isAllChoose = chooseNumber === this.checkbox.length
      this.isIndeterminate = chooseNumber > 0 && chooseNumber < this.checkbox.length
      if (this.checkbox[scope.$index]) {
        this.checkList.push(scope.row)
      } else {
        this.checkList = this.checkList.filter((val) => {
          return val.id !== scope.row.id
        })
      }
    },
    btnClick () {
      let flag = true
      this.checkbox.forEach((val, index) => {
        if (val) {
          // 验证每一项
          this.$refs.dataForm.validateField(`tableData.${index}.description`, (errorMes) => {
            if (errorMes !== '') {
              flag = false
            }
          })
        }
      })
      if (flag) {
        alert('提交成功')
      }
    }
  }
}
</script>

<style>
</style>
