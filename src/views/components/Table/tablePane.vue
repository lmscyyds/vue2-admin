<template>
  <div>
    <el-card class="mt10">
      <div v-if="dataSource.tool" class="tool">
        <el-button
          v-for="(item) in dataSource.tool"
          :key="item.key"
          :style="{'background':item.bgColor,borderColor:item.bgColor}"
          :type="item.type || 'primary'"
          class="filter-item"
          @click="item.handleClick(item.name,$event)"
        >
          {{ item.name }}
        </el-button>
      </div>
      <el-table
        ref="table"
        v-loading="dataSource.loading"
        :class="{ 'no-data': !dataSource.data || !dataSource.data.length }"
        :data="dataSource.data"
        style="width: 100%;"
        @row-click="getRowData"
        :height="height + 'px'"
        @selection-change="dataSource.handleSelectionChange"
      >
        <!-- 是否有多选 -->
        <el-table-column
          v-if="dataSource.isSelection"
          :selectable="dataSource.selectable"
          :width="dataSource.selectionWidth || 50"
          align="center"
          type="selection"
        />
        <!-- 是否需要序号 -->
        <el-table-column
          v-if="dataSource.isIndex"
          align="center"
          label="序号"
          type="index"
          width="55"
        />

        <template v-for="item in dataSource.cols">
          <!-- 表格的列展示 特殊情况处理 比如要输入框 显示图片 -->
          <el-table-column
            v-if="item.isTemplate"
            :key="item.prop"
            v-bind="item"
          >
            <template slot-scope="scope">
              <!-- 比如要输入框 显示图片等等 自己定义 -->
              <slot :name="item.prop" :scope="scope" />
            </template>
          </el-table-column>
          <!-- 需要特殊颜色显示字体-->
          <el-table-column
            v-if="item.isSpecial"
            :key="item.prop"
            align="center"
            v-bind="item"
          >
            <template slot-scope="scope">
            <span
              :class="item.isSpecialClass(scope.row[scope.column.property])"
            >{{ item.isSpecial(scope.row[scope.column.property]) }}</span>
            </template>
          </el-table-column>
          <!-- 需要带图标的某列，带回调事件-->
          <el-table-column
            v-if="item.isIcon"
            :key="item.prop"
            align="center"
            v-bind="item"
          >
            <template slot-scope="scope">
            <span>
              <span>{{ item.filter(scope.row[scope.column.property]) }}</span>
              <i
                v-if="item.icon"
                :class="[item.icon(scope.row[scope.column.property]),'icon-normal']"
                @click="item.handlerClick(scope.row)"
              />
            </span>
              <!-- 比如要输入框 显示图片等等 自己定义 -->
              <slot :name="item.prop" :scope="scope" />
            </template>
          </el-table-column>
          <!-- 图片带tooltip -->
          <el-table-column
            v-if="item.isImagePopover"
            :key="item.prop"
            align="center"
            v-bind="item"
          >
            <template slot-scope="scope">
              <el-popover
                placement="right"
                title=""
                trigger="hover"
              >
                <img
                  :src="scope.row[scope.column.property]+'?x-oss-process=image/quality,q_60'"
                  alt=""
                  class="image-popover"
                >
                <img
                  slot="reference"
                  :src="scope.row[scope.column.property]+'?x-oss-process=image/quality,q_10'"
                  alt=""
                  class="reference-img"
                >
              </el-popover>
            </template>
          </el-table-column>
          <!-- 大部分适用 -->
          <el-table-column
            v-if="!item.isImagePopover && !item.isTemplate && !item.isSpecial&&!item.isIcon"
            :key="item.prop"
            align="center"
            show-overflow-tooltip
            v-bind="item.isCodeTableFormatter ? Object.assign({ formatter: item.isCodeTableFormatter }, item) : item"
          />
        </template>
        <!-- 是否有操作列 -->
        <!-- 没有数据时候不固定列 -->
        <el-table-column
          v-if="dataSource.isOperation"
          :label="dataSource.operation.label"
          :show-overflow-tooltip="dataSource.operation.overflowTooltip"
          :width="dataSource.operation.width"
          align="center"
          class-name="handle-td"
          label-class-name="tc"
          style="margin-right:20px"
          v-bind="dataSource.data && dataSource.data.length ? { fixed: 'right' } : null"
        >
          <!-- UI统一一排放3个，4个以上出现更多 -->
          <template slot-scope="scope">
            <!-- 三个一排的情况，去掉隐藏的按钮后的长度 -->
            <template v-if="dataSource.operation.data.length > 0">
              <div class="btn">
                <div v-for="(item) in dataSource.operation.data" :key="item.label">
                  <template v-if="item.type!=='icon'">
                    <el-button
                      :type="item.type?item.type:''"
                      size="mini"
                      v-bind="item"
                      @click.native.prevent="item.handleRow(scope.$index, scope.row, item.label)"
                    >
                      {{ item.label }}
                    </el-button>
                  </template>
                  <template v-else>
                    <i
                      :class="[icon,item.icon]"
                      v-bind="item"
                      @click="item.handleRow(scope.$index, scope.row, item.label)"
                    />
                  </template>
                </div>
              </div>
            </template>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-card class="mt10 el-card-page">
      <div class="page">
        <el-pagination
          v-if="dataSource.pageData.total>0"
          :current-page="dataSource.pageData.pageNum"
          :page-size="dataSource.pageData.pageSize"
          :page-sizes="dataSource.pageData.pageSizes?dataSource.pageData.pageSizes:[5,10,15,20]"
          :total="dataSource.pageData.total"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </el-card>
  </div>
</template>

<script>
export default {
  // 接收父组件传递过来的值
  props: {
    //  表格数据和表格部分属性的对象
    // eslint-disable-next-line vue/require-default-prop
    dataSource: {
      type: Object
    }
  },
  data() {
    return {
      height: ''
    }
  },
  created() {
    const innerHeight = window.innerHeight
    const filter_height = this.$store.getters.filterHeight
    this.$nextTick(() => {
      const el_card_page = document.getElementsByClassName('el-card-page')[0].scrollHeight
      const tool = document.getElementsByClassName('tool')[0].scrollHeight
      const el_table__header_wrapper = document.getElementsByClassName('el-table__header-wrapper')[0].scrollHeight
      this.height = innerHeight - 50 - 34 - el_card_page - filter_height - 20 - 40 - tool - el_table__header_wrapper - 10
    })
  },
  watch: {
    'dataSource.cols': { // 监听表格列变化
      deep: true,
      handler() {
        // 解决表格列变动的抖动问题
        this.$nextTick(this.$refs.table.doLayout)
      }
    }
  },
  methods: {
    handleAdd(name) {
      console.log(name)
      this.$emit('toolMsg', name)
    },
    handleRow(index, row, lable) {
      console.log(index, row, lable)
    },
    handleSizeChange(val) {
      this.$emit('changeSize', val)
      console.log(`每页 ${val} 条`)
    },
    handleCurrentChange(val) {
      this.$emit('changeNum', val)
      console.log(`当前页: ${val}`)
    },
    // 点击行即可选中
    getRowData(row) {
      this.$refs.table.toggleRowSelection(row)
    }
  }
}
</script>
<style lang="scss" scoped>
.page {
  display: flex;
  justify-content: center;
}

.btn {
  display: flex;
  justify-content: center;
}

.btn div {
  margin-left: 5px;
}

.reference-img {
  width: 40px;
  height: 40px;
  background-size: 100% 100%;
  border-radius: 4px;
}

.image-popover {
  width: 200px;
  height: 200px;
  background-size: 100% 100%;
}

.icon {
  width: 25px;
  font-size: 20px;
  font-weight: bold;
}

.mt10{
  margin-top: 10px;
}
</style>
