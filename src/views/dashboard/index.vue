<template>
  <div className="app-container">
    <filter-pane :filter-data="filterData" @filterMsg="filterMsg" />
    <table-pane
      :data-source="dataSource"
      @changeNum="changeNum"
      @changeSize="changeSize"
    />
    <!-- <view-code :dialog-view-code="dialogViewCode" :dialog-view-code-data="dialogViewCodeData" @childMsg="childMsg" /> -->
  </div>
</template>
<script>
import filterPane from '../components/Table/filterPane'
import tablePane from '../components/Table/tablePane'
import { getList } from '@/api/table'

export default {
  name: '',
  components: { filterPane, tablePane },
  data() {
    return {
      // 搜索栏配置
      filterData: {
        timeSelect: true, // 时间组件
        elinput: [{
          key: 'userId',
          name: '脸影号',
          width: 230
        }],
        elselect: [{
          name: '默认排序',
          width: 120,
          key: 'sortType',
          option: [
            {
              key: 1,
              value: '默认排序'
            },
            {
              key: 2,
              value: '金币降序'
            },
            {
              key: 3,
              value: '余额降序'
            }
          ]
        }]
      },
      // 表格配置
      dataSource: {
        tool: [
          {
            name: '一键复制手机号', // 按钮名称
            type: '', // 默认primary，（非必须）
            key: 1, // 唯一标识符
            permission: 2010106, // 权限点
            bgColor: '#67c23a', // 自定义背景色（非必须）
            handleAdd: this.handleAdd// 回调事件
          }
        ], // 表格上方工具栏
        data: [], // 表格数据
        cols: [
          // 普通列
          {
            label: 'title', // 列名
            prop: 'title', // 字段名称
            width: 100 // 列宽度
          },
          // 带过滤器列
          {
            label: 'author',
            prop: 'author',
            // isCodeTableFormatter: function(val) { // 过滤器
            //   if (val.subtitle === 0) {
            //     return '无'
            //   } else {
            //     return val.subtitle
            //   }
            // },
            width: 100
          },
          // 事件列timeFormat自己写
          {
            label: 'pageviews',
            prop: 'pageviews',
            // isCodeTableFormatter: function(val) {//时间过滤器
            //   return timeFormat(val.createTime)
            // },
            width: 150
          },
          // 普通列字体颜色改变
          {
            label: 'display_time',
            prop: 'display_time',
            // isTemplate: function(val) {
            //   if (val === 1) {
            //     return '禁言中'
            //   } else {
            //     return '已解禁'
            //   }
            // },
            // isTemplateClass: function(val) {
            //   if (val === 1) {
            //     return 'color-red'
            //   } else {
            //     return 'color-green'
            //   }
            // }
          },
          // {
          //   label: '目标类型',
          //   prop: 'targetType',
          //   permission: '', // 需要的自己加权限点
          //   isIcon: true,
          //   filter: function(val) {
          //     if (val === 4) {
          //       return '特定用户'
          //     } else if (val === 3) {
          //       return '新注册用户'
          //     } else if (val === 2) {
          //       return '标签用户'
          //     } else if (val === 1) {
          //       return '全部用户'
          //     }
          //   },
          //   icon: function(val) {
          //     if (val === 4) {
          //       return 'el-icon-mobile'
          //     } else {
          //       return false
          //     }
          //   },
          //   handlerClick: this.handlerClick
          // }

        ], // 表格的列数据
        handleSelectionChange: this.handleSelectionChange, // 多选回调
        isSelection: false, // 表格有多选时设置
        selectable: function(val) { // 禁用部分行多选（非必须）
          if (val.isVideoStatus === 1) {
            return false
          } else {
            return true
          }
        },
        isOperation: true, // 表格有操作列时设置,无操作列为false
        isIndex: true, // 列表序号
        loading: true, // loading
        pageData: {
          total: 0, // 总条数
          pageSize: 10, // 每页数量
          pageNum: 1 // 页码
        },
        operation: {
          // 表格有操作列时设置
          label: '操作', // 列名
          width: '100', // 根据实际情况给宽度
          data: [
            {
              type: 'primary',
              label: '解封', // 操作名称
              permission: '8010201', // 后期这个操作的权限，用来控制权限
              handleRow: this.handleRow
            }
          ]
        }
      },
      msg: {}, // 储存搜索条件
      selected: []// 多选必须，储存已选内容
    }
  },
  created() {
    this.getList()
  },
  methods: {
    // 获取列表数据
    getList() {
      const data = {
        pageSize: this.dataSource.pageData.pageSize,
        pageNum: this.dataSource.pageData.pageNum
      }
      if (this.msg.userId) {
        var reg = /^\d{9,10}$/
        if (!reg.test(this.msg.userId)) {
          this.$message({
            type: 'error',
            message: '请输入正确的脸影号'
          })
          return
        }
        data.userId = this.msg.userId
      }
      if (this.msg.sortType) {
        data.sortType = this.msg.sortType
      }
      this.dataSource.loading = true
      getList(data).then(res => {
        this.dataSource.loading = false
        if (res.data.total > 0) {
          this.dataSource.pageData.total = res.data.total
          this.dataSource.data = res.data.items
        } else {
          this.dataSource.data = []
          this.dataSource.pageData.total = 0
        }
      })
    },
    // 搜索栏事件
    filterMsg(msg) {
      this.msg = msg
      this.getList()
    },
    // 子组件通信
    childMsg(msg) {
    },
    // 改变每页数量
    changeSize(size) {
      this.dataSource.pageData.pageSize = size
      this.getList()
    },
    // 改编页码
    changeNum(pageNum) {
      this.dataSource.pageData.pageNum = pageNum
      this.getList()
    },
    // 表格上方操作行
    handleAdd(name, row) {
      console.log(name, row)
    },
    // 多选回调
    handleSelectionChange(val) {
      this.selected = val
    },
    // 表格操作栏事件
    handleRow(index, row, lable) {
      console.log(index, row, lable)
    },
    isSon(row) {
      console.log(row)
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
