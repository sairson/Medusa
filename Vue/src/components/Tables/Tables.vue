<script>
export default {
  props: {
    columns: {
      type: Array,
      default: () => {
        return []
      }
    },
    tableData: {
      type: Array,
      default: () => {
        return []
      }
    },
    total: {
      type: Number,
      default: 0
    },
    rowKey: {
      type: String | Function,
      default: 'key' | function () { }
    },
    showPagination: {//是否分页
      type: Boolean,
      default: false
    },
    showExpandedRowKeys: {//是否展开默认行
      type: Boolean,
      default: false
    },
    scrollTable: {
      type: Object,
      default: () => {
        return { x: 1600, y: 450 }
      }
    },
    ExpandedRowRenderCallback: {
      tyep: Function,
      default: () => {
        return function () { }
      }
    },
    ExpandIconCallback: {
      tyep: Function,
      default: () => {
        return function () { }
      }
    },
    ShowTotal: {
      tyep: Function,
      default: () => {
        return function () { }
      }
    }
  },
  data () {
    return {
      pagination: {
        showQuickJumper: true,
        position: 'bottom',
        defaultPageSize: 100,
        pageSizeOptions: ['100'],
        hideOnSinglePage: false,
        showTotal: (total, range) => {
          const _this = this
          const Dom = <span style="display: flex;align-items: center;padding-right:10px;font-size:20px" v-show={total == 0 ? false : true}>共<span style="color:#51c51a;padding:0 10px 0 10px"><a-statistic value={total} valueStyle={{ color: '#51c51a' }} >
          </a-statistic></span>条</span>
          const callback = _this.ShowTotal(Dom, total, range)
          if (callback) {
            return callback
          }
          else return Dom
        },
        // itemRender: (page, type, originalElement) => {
        //   console.log(page, type, originalElement)
        //   if (type === 'prev') {
        //     return <a>Previous</a>;
        //   } else if (type === 'next') {
        //     return <a>Next</a>;
        //   }
        //   return originalElement;
        // },
        total: 0
      },
      expandedRowKeys: []
    }
  },
  methods: {
    handleChange (pagination, filters, sorter, { currentDataSource }) {
      this.$emit('change', pagination)
    },
    handleExpandedRowRender (record, index, indent, expanded) {
      const _this = this
      const callback = _this.ExpandedRowRenderCallback(record)
      return callback
    },
    handleFooter (currentPageData) {
      const _this = this
      return <div style="text-align:left" v-show={_this.total == 0 ? false : true}>总共<span style="color:#51c51a">{_this.total}</span>条数据</div>
    },
    handleExpandedRowKeys () {//全部展开
      const _this = this
      _this.$nextTick(() => {
        _this.expandedRowKeys = _this.tableData.map((item, i) => (typeof (_this.rowKey) === 'function') ? i : item[_this.rowKey])
      })
    },
    handleExpandIcon (props) {
      const _this = this
      const callback = _this.ExpandIconCallback(props)
      return callback
    }
  },
  watch: {
    total: {
      handler: function (now, old) {
        const _this = this
        _this.pagination.total = now
      },
      deep: true
    },
    tableData: {
      handler: function (now, old) {
        const _this = this
        if (_this.showExpandedRowKeys) _this.handleExpandedRowKeys()
        else _this.expandedRowKeys = []
      },
      deep: true
    }
  },
  render: function (h) {
    const _this = this
    if (_this.expandedRowKeys.length == 0) {
      return h('a-table', {
        props: {
          columns: _this.columns,
          dataSource: _this.tableData,
          scroll: _this.scrollTable,
          pagination: _this.showPagination ? false : _this.pagination,
          total: _this.total,
          rowKey: _this.rowKey,
        },
        on: {
          change: (pagination, filters, sorter, { currentDataSource }) => _this.handleChange(pagination, filters, sorter, { currentDataSource }),
        }
      })
    }
    else {
      return h('a-table', {
        props: {
          columns: _this.columns,
          dataSource: _this.tableData,
          scroll: _this.scrollTable,
          pagination: _this.pagination,
          total: _this.total,
          rowKey: _this.rowKey,
          expandedRowKeys: _this.expandedRowKeys,
          // footer: (currentPageData) => _this.handleFooter(currentPageData),
          expandedRowRender: (record, index, indent, expanded) => _this.handleExpandedRowRender(record, index, indent, expanded),
          expandIcon: (props) => _this.handleExpandIcon(props),
        },
        on: {
          change: (pagination, filters, sorter, { currentDataSource }) => _this.handleChange(pagination, filters, sorter, { currentDataSource }),
          expandedRowsChange: (expandedRowKeys) => {
            _this.expandedRowKeys = expandedRowKeys
          }
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
// ::v-deep .ant-table {
//   minheight: ;
// }
/*定义整体的宽度*/
::v-deep .ant-table-body::-webkit-scrollbar {
  height: 5px;
  width: 5px;
}

/*定义滚动条轨道*/
::v-deep .ant-table-body::-webkit-scrollbar-track {
  border-radius: 2px;
}

/*定义滑块*/
::v-deep .ant-table-body::-webkit-scrollbar-thumb {
  border-radius: 2px;
  background: rgba(0, 255, 42, 0.5);
}
</style>