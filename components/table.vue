<template>
<div style="width: 100%; font-size: 12px;">
  <el-table
    ref="main-table"
    v-loading="isLoading"
    :data="tableBody.content"
    style="width: 100%; text-align: left;"
    @selection-change="handleSelectRows"
    :stripe="true"
    highlight-current-row
    @current-change="handleCurrentRowChange"
  >
    <el-table-column v-if="option.selectable" type="selection"></el-table-column>
    <el-table-column
      v-for="header in headers"
      :key="header._id"
      :label="header.t_tab_name"
      :prop="header.t_tab_key"
      :render-header="renderCustomHeader"
      min-width="100"
      :sortable="false"
    >
    </el-table-column>
    <el-table-column
      v-if="option.hasOperateColumn"
      fixed="right"
      label=""
      width="180"
    >
      <template slot-scope="scope">
        <div>
          <el-button
            v-for="btn in option.operates"
            :key="btn.id"
            class="el-button el-button--default el-button--mini"
            :class="[btn.css]"
            @click="btn.handleClick(scope.row, $event)"
          >
            {{btn.label}}
          </el-button>
        </div>
      </template>
    </el-table-column>
  </el-table>
  <div class="qz-table-body-pagination-container">
    <el-pagination
      v-if="option.hasPagination"
      @size-change="handleChangePageSize"
      @current-change="handleChangeCurrentPage"
      :current-page="tableBody.pageNum"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="tableBody.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="tableBody.total"
    >
    </el-pagination>
  </div>
  <el-dialog
    :title="currentFilterInfo.label"
    :visible.sync="dateDialogVisible"
  >
    <div class="date-pick-container">
      <el-date-picker
        v-model="filterConditionByTTabKey[currentFilterInfo.key]"
        type="datetimerange"
        range-separator="TO"
        start-placeholder="Begin"
        end-placeholder="End"
        @change="handleDateFilterChanged"
        :editable="false"
      >
      </el-date-picker>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="handleDateFilterChanged">OK</el-button>
    </span>
  </el-dialog>
</div>
</template>

<style scoped>
  .qz-table-body-pagination-container {
    margin: 26px 0;
    display: flex;
    justify-content: flex-end;
    align-items: center;
  }

  .date-pick-container {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: flex-start;
  }
</style>

<script>
export default {
  props: {
    option: {
      type: Object,
      default: function () {
        return {
          selectable: false,
          sortable: true,
          hasOperateColumn: false,
          operates: [],
          startPage: 1,
          pageSize: 10,
          hasPagination: true
        }
      }
    },
    handleSelectRows: {
      type: Function,
      default: function (rows) {
        console.warn('[qz-table] - You should not use the default function (handleSelectRows) directly.')
      }
    },
    handleCurrentRowChange: {
      type: Function,
      default: function (currentRow) {
        console.warn('[qz-table] - You should not use the default function (handleCurrentChange) directly.')
      }
    },
    loadRemoteData: {
      type: Function,
      default: function (setTableData, setLoading, currentPage, pageSize, filterConditionByTTabKey) {
        throw new Error('[qz-table] - Please implement the loadRemoteData method')
      }
    }
  },
  data () {
    return {
      tableBody: {
        total: 0,
        pageNum: 1,
        pages: 0,
        pageSize: 10,
        startRow: 1,
        endRow: 10,
        content: []
      },
      headers: [],
      isFilterBarShow: false,
      filterConditionByTTabKey: {},
      isLoading: false,
      dateDialogVisible: false,
      currentFilterInfo: {},
      filterContentByTTabKey: {},
      selfId: '',
      sort_order: 'descending'
    }
  },
  methods: {
    setActiveTableForSelf (e) {
      e && e.stopPropagation()
      window && (window.qz_tb_payload = {
        ...window.qz_tb_payload,
        activeId: this.selfId
      })
    },
    setTableData (headersRequestData, bodyRequestData, filterContentByTTabKey) {
      this.headers = [ ...headersRequestData ]
      this.tableBody = { ...bodyRequestData }
      this.filterContentByTTabKey = { ...filterContentByTTabKey }
      this.setActiveTableForSelf()
      this.isLoading = false
    },
    setLoading (isLoading = true) {
      this.isLoading = isLoading
    },
    handleClearQueryCondition (e) {
      e && e.preventDefault()
      if (e.which === 27 && window && window.qz_tb_payload && window.qz_tb_payload.activeId === this.selfId) {
        this.filterConditionByTTabKey = {}
        this.isFilterBarShow = false
        const setTableData = this.setTableData.bind(this)
        const setLoading = this.setLoading.bind(this)
        this.dateDialogVisible = false
        this.loadRemoteData(setTableData, setLoading, 1, this.tableBody.pageSize, this.filterConditionByTTabKey)
        return false
      }
    },
    handleChangeCurrentPage (currentPage) {
      const setTableData = this.setTableData.bind(this)
      const setLoading = this.setLoading.bind(this)
      this.loadRemoteData(setTableData, setLoading, currentPage, this.tableBody.pageSize, this.filterConditionByTTabKey)
    },
    handleChangePageSize (pageSize) {
      const setTableData = this.setTableData.bind(this)
      const setLoading = this.setLoading.bind(this)
      this.loadRemoteData(setTableData, setLoading, 1, pageSize, this.filterConditionByTTabKey)
    },
    handleDateFilterChanged () {
      const setTableData = this.setTableData.bind(this)
      const setLoading = this.setLoading.bind(this)
      this.dateDialogVisible = false
      this.loadRemoteData(setTableData, setLoading, 1, this.tableBody.pageSize, this.filterConditionByTTabKey)
    },
    handleInputFilterChanged () {
      const setTableData = this.setTableData.bind(this)
      const setLoading = this.setLoading.bind(this)
      this.inputDialogVisible = false
      this.loadRemoteData(setTableData, setLoading, 1, this.tableBody.pageSize, this.filterConditionByTTabKey)
    },
    handleSelectFilterChanged () {
      const setTableData = this.setTableData.bind(this)
      const setLoading = this.setLoading.bind(this)
      this.selectDialogVisible = false
      this.loadRemoteData(setTableData, setLoading, 1, this.tableBody.pageSize, this.filterConditionByTTabKey)
    },
    renderCustomHeader (h, { column, $index }) {
      const outerContainerStyle = {
        'display': 'inline-flex',
        'justify-content': 'space-around',
        'align-items': 'flex-start',
        'flex-direction': 'column',
        'width': '100%',
        'padding-left': 0
      }

      const headlineContainerStyle = {
        'display': 'inline-flex',
        'justify-content': 'space-between',
        'align-items': 'center',
        'flex-direction': 'row',
        'width': '100%',
        'padding': 0,
        'margin': 0
      }

      const inputContainerStyle = {
        'display': 'block',
        'width': '100%',
        'padding': 0,
        'margin': 0
      }

      const tableTitleStyle = {
        display: 'inline',
        'font-weight': '500',
        overflow: 'hidden',
        'text-overflow': 'ellipsis',
        'white-space': 'nowrap'
      }

      const sortBtnStyle = {
        'margin-left': '6px', 
        display: 'inline-block',
        cursor: 'pointer'
      }

      const self = this

      const filterContentByTTabKey = this.filterContentByTTabKey
      const currentColumnOriginalData = this.headers && this.headers[this.headers.findIndex(header => column.property === header.t_tab_key)]
      const filterType = currentColumnOriginalData && currentColumnOriginalData.t_tab_flag

      const filiteValue = (this.filterConditionByTTabKey && this.filterConditionByTTabKey[column.property]) || ''

      const switchDateDialogVisible = () => {
        this.currentFilterInfo = {
          key: column.property,
          label: column.label
        }
        this.dateDialogVisible = true
        this.setActiveTableForSelf()
      }

      const setActiveTableForSelf = this.setActiveTableForSelf.bind(this)

      const switchFilterBarShow = e => {
        e.stopPropagation()
        this.isFilterBarShow = !this.isFilterBarShow
      }

      const getDate = dateString => {
        const temp = new Date(dateString)
        return temp.getFullYear() + '-' + (temp.getMonth() + 1) + '-' + temp.getDate() + ' ' + (temp.getHours() < 10 ? '0' + temp.getHours() : temp.getHours()) + ':' + (temp.getMinutes() < 10 ? '0' + temp.getMinutes() : temp.getMinutes())
      }

      const getDatesText = (dateList, separator = ' To ') => {
        if (!dateList) return ''
        return dateList.map(dateString => getDate(dateString)).join(separator)
      }

      const handleInputChange = value => {
        this.filterConditionByTTabKey[column.property] = value
      }

      const handleSelectChange = value => {
        this.filterConditionByTTabKey[column.property] = value
        this.handleSelectFilterChanged()
      }

      const handleFilterClick = e => {
        e && e.stopPropagation()
      }

      const handleManualSort = () => {
        this.$refs['main-table'].sort(column.property, this.sort_order === 'descending' ? 'ascending' : 'descending')
        this.sort_order = (this.sort_order === 'descending' ? 'ascending' : 'descending')
      }

      if (filterType === 'input') {
        return (
          <div style={outerContainerStyle}>
            <div style={headlineContainerStyle} onClick={handleFilterClick} onDblclick={switchFilterBarShow}>
              <h3 style={tableTitleStyle}>{ column.label }</h3>
              <div>
                {
                  this.isFilterBarShow
                    ? <i class="el-icon-caret-top" style="cursor: pointer;" onClick={switchFilterBarShow}/> : <i class="el-icon-caret-bottom" style="cursor: pointer;" onClick={switchFilterBarShow}/>
                }
                {self.option.sortable && <i class="el-icon-sort" style={sortBtnStyle} onClick={handleManualSort}/>}
              </div>
            </div>
            <el-collapse-transition>
              {
                this.isFilterBarShow &&
                  (<div style={inputContainerStyle}>
                    <el-input
                      size='mini'
                      value={filiteValue}
                      onFocus={setActiveTableForSelf}
                      onInput={handleInputChange}
                      onChange={self.handleInputFilterChanged}
                      onBlur={self.handleInputFilterChanged}
                    ></el-input>
                  </div>)
              }
            </el-collapse-transition>
          </div>
        )
      }

      if (filterType === 'date') {
        return (
          <div style={outerContainerStyle}>
            <div style={headlineContainerStyle} onClick={handleFilterClick} onDblclick={switchFilterBarShow}>
              <h3 style={tableTitleStyle}>{ column.label }</h3>
              <div>
                {
                  this.isFilterBarShow
                    ? <i class="el-icon-caret-top" style="cursor: pointer;" onClick={switchFilterBarShow}/> : <i class="el-icon-caret-bottom" style="cursor: pointer;" onClick={switchFilterBarShow}/>
                }
                {self.option.sortable && <i class="el-icon-sort" style={sortBtnStyle} onClick={handleManualSort}/>}
              </div>
            </div>
            <el-collapse-transition>
              {
                this.isFilterBarShow &&
                  (<div style={inputContainerStyle}>
                    <el-input
                      style="padding: 0 6px; cursor: pointer;"
                      size='mini'
                      value={getDatesText(filiteValue)}
                      suffix-icon="el-icon-date"
                      onFocus={switchDateDialogVisible}
                    ></el-input>
                  </div>)
              }
            </el-collapse-transition>
          </div>
        )
      }

      if (filterType === 'select') {
        return (
          <div style={outerContainerStyle}>
            <div style={headlineContainerStyle} onClick={handleFilterClick} onDblclick={switchFilterBarShow}>
              <h3 style={tableTitleStyle}>{ column.label }</h3>
              <div>
                {
                  this.isFilterBarShow
                    ? <i class="el-icon-caret-top" style="cursor: pointer;" onClick={switchFilterBarShow}/> : <i class="el-icon-caret-bottom" style="cursor: pointer;" onClick={switchFilterBarShow}/>
                }
                {self.option.sortable && <i class="el-icon-sort" style={sortBtnStyle} onClick={handleManualSort}/>}
              </div>
            </div>
            <el-collapse-transition>
              {
                this.isFilterBarShow &&
                  (<div style={inputContainerStyle}>
                    <el-select size="mini" placeholder='' value={filiteValue} onFocus={setActiveTableForSelf} onChange={handleSelectChange}> 
                      {filterContentByTTabKey[column.property].map(v => (<el-option key={v} value={v}>{v}</el-option>))}
                    </el-select>
                  </div>)
              }
            </el-collapse-transition>
          </div>
        )
      }

      return (
        <div style={outerContainerStyle}>
          <div style={headlineContainerStyle} onClick={handleFilterClick} onDblclick={switchFilterBarShow}>
            <h3 style={tableTitleStyle}>{ column.label }</h3>
            <div>
              {
                this.isFilterBarShow
                  ? <i class="el-icon-caret-top" style="cursor: pointer;" onClick={switchFilterBarShow}/> : <i class="el-icon-caret-bottom" style="cursor: pointer;" onClick={switchFilterBarShow}/>
              }
              {self.option.sortable && <i class="el-icon-sort" style={sortBtnStyle} onClick={handleManualSort}/>}
            </div>
          </div>
          <el-collapse-transition>
            {
              this.isFilterBarShow &&
                (<div style={inputContainerStyle}>
                  <el-input size='mini' disabled value={filiteValue} clearable></el-input>
                </div>)
            }
          </el-collapse-transition>
        </div>
      )
    }
  },
  created () {
    const setTableData = this.setTableData.bind(this)
    const setLoading = this.setLoading.bind(this)
    this.selfId = Date.now() + 'qz'
    window && (window.qz_tb_payload = {
      ...window.qz_tb_payload,
      activeId: this.selfId
    })
    this.loadRemoteData(setTableData, setLoading, this.option.startPage, this.option.pageSize, this.filterConditionByTTabKey)
    $('body').on('keyup', this.handleClearQueryCondition)
  },
  destroyed () {
    $('body').off('keyup', 'body', this.handleClearQueryCondition)
  }
}
</script>

