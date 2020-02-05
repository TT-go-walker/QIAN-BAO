<template>
  <a-card :bordered="false">
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="48">
          <a-col :md="8" :sm="24">
            <a-form-item label="用户地址">
              <a-input v-model="queryParam.address" placeholder=""/>
            </a-form-item>
          </a-col>
          <a-col :md="8" :sm="24">
            <a-form-item label="时间">
              <a-range-picker @change="queryTimeChange" />
              <!--<a-date-picker v-model="queryParam.date" style="width: 100%" placeholder="请输入更新日期"/>-->
            </a-form-item>
          </a-col>
          <a-col :md="8" :sm="24">
            <a-form-item label="对方地址">
              <a-input v-model="queryParam.adverseAddress" placeholder=""/>
            </a-form-item>
          </a-col>
          <template v-if="advanced">
            <a-col :md="8" :sm="24">
              <a-form-item label="转入/出">
                <a-select v-model="queryParam.type" placeholder="请选择" default-value="0">
                  <a-select-option value="0">全部</a-select-option>
                  <a-select-option value="1">转入</a-select-option>
                  <a-select-option value="2">转出</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="商户账号">
                <a-input v-model="queryParam.merchAccount" placeholder="" />
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="txid">
                <a-input v-model="queryParam.txid" placeholder="请输入txid" />
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="事项">
                <a-select v-model="queryParam.matter" placeholder="请选择" default-value="0">
                  <a-select-option value="0">全部</a-select-option>
                  <a-select-option value="1">提现</a-select-option>
                  <a-select-option value="2">充值</a-select-option>
                  <a-select-option value="3">合约转入</a-select-option>
                  <a-select-option value="4">合约转出</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="币种">
                <a-select v-model="queryParam.currency" placeholder="请选择" default-value="0">
                  <a-select-option v-for="(item,index) in currencyMap" :key="index" :value="item.id">{{ item.text }}</a-select-option>
                  <!--<a-select-option value="0">全部</a-select-option>
                  <a-select-option value="1">关闭</a-select-option>
                  <a-select-option value="2">运行中</a-select-option>-->
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="状态">
                <a-select v-model="queryParam.status" placeholder="请选择" default-value="0">
                  <a-select-option value="0">全部</a-select-option>
                  <a-select-option value="1">进行中</a-select-option>
                  <a-select-option value="2">成功</a-select-option>
                  <a-select-option value="3">失败</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="扫币">
                <a-select v-model="queryParam.status2" placeholder="请选择" default-value="0">
                  <a-select-option value="0">全部</a-select-option>
                  <a-select-option value="1">完成</a-select-option>
                  <a-select-option value="2">未完成</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="appid">
                <a-input v-model="queryParam.appid" placeholder="请输入appid" />
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="!advanced && 8 || 24" :sm="24">
            <span class="table-page-search-submitButtons" :style="advanced && { float: 'right', overflow: 'hidden' } || {} ">
              <a-button type="primary" @click="$refs.table.refresh(true)">查询</a-button>
              <a-button style="margin-left: 8px" @click="() => queryParam = {}">重置</a-button>
              <a @click="toggleAdvanced" style="margin-left: 8px">
                {{ advanced ? '收起' : '展开' }}
                <a-icon :type="advanced ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <div class="table-operator">
      <!--<a-button type="primary" icon="plus" @click="$refs.createModal.add()">新建</a-button>
      <a-button type="dashed" @click="tableOption">{{ optionAlertShow && '关闭' || '开启' }} alert</a-button>-->
      <a-dropdown v-action:edit v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1"><a-icon type="delete" />删除</a-menu-item>
          <!-- lock | unlock -->
          <a-menu-item key="2"><a-icon type="lock" />锁定</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作 <a-icon type="down" />
        </a-button>
      </a-dropdown>
    </div>

    <s-table
      ref="table"
      size="default"
      rowKey="key"
      :columns="columns"
      :data="loadData"
      :alert="options.alert"
      :rowSelection="options.rowSelection"
      showPagination="auto"
      :expandRowByClick="true"
    >
      <p slot="expandedRowRender" slot-scope="record" style="margin: 0;">
        txid : {{ record.txid }} <br>
        对方地址 : {{ record.adverseAddress }} <br>
        商户账号 : {{ record.merchAccount }} <br>
        appid : {{ record.appid }} <br>
        矿工费 : {{ record.salary }} <br>
        扫币 : {{ record.status2 | status2Filter }} <br>
      </p>
      <span slot="currency" slot-scope="text">
        <!--<a-badge :status="text | statusTypeFilter" :text="text | statusFilter" />-->
        {{ text | currencyFilter }}
      </span>
      <span slot="type" slot-scope="text">
        {{ text | typeFilter }}
      </span>
      <span slot="matter" slot-scope="text">
        {{ text | matterFilter }}
      </span>
      <span slot="status" slot-scope="text">
        {{ text | statusFilter }}
      </span>
      <span slot="status2" slot-scope="text">
        {{ text | status2Filter }}
      </span>
      <!--<span slot="description" slot-scope="text">
        <ellipsis :length="4" tooltip>{{ text }}</ellipsis>
      </span>-->

      <span slot="action">
        <template>
          <a>详情</a>
        </template>
      </span>
    </s-table>
    <create-form ref="createModal" @ok="handleOk" />
    <step-by-step-modal ref="modal" @ok="handleOk"/>
  </a-card>
</template>

<script>
import moment from 'moment'
import { STable, Ellipsis } from '@/components'
import StepByStepModal from './modules/StepByStepModal'
import CreateForm from './modules/CreateForm'
import { getRoleList, getServiceList } from '@/api/manage'

const typesMap = {
  1: {
    status: 'processing',
    text: '转入'
  },
  2: {
    status: 'success',
    text: '转出'
  }
}
const mattersMap = {
  1: {
    status: 'processing',
    text: '充值'
  },
  2: {
    status: 'success',
    text: '提现'
  },
  3: {
    status: 'processing',
    text: '合约转入'
  },
  4: {
    status: 'success',
    text: '合约转出'
  }
}
const status2Map = {
  1: {
    status: 'processing',
    text: '完成'
  },
  2: {
    status: 'success',
    text: '未完成'
  }
}
const statusMap = {
  1: {
    status: 'processing',
    text: '进行中'
  },
  2: {
    status: 'success',
    text: '成功'
  },
  3: {
    status: 'error',
    text: '失败'
  }
}
/* -------币种----------- */
const currencyMap = [
  {
    id: 0,
    text: 'BTC'
  },
  {
    id: 1,
    text: 'USD'
  },
  {
    id: 2,
    text: 'USDT'
  },
  {
    id: 3,
    text: 'CNY'
  }
]
export default {
  name: 'SiteManageAsset',
  components: {
    STable,
    Ellipsis,
    CreateForm,
    StepByStepModal
  },
  data () {
    return {
      mdl: {},
      // 高级搜索 展开/关闭
      advanced: false,
      // 查询参数
      queryParam: {},
      // 表头
      columns: [
        {
          title: '时间',
          dataIndex: 'updatedAt',
          sorter: true
        },
        {
          title: '商户地址',
          dataIndex: 'address'
        },
        {
          title: '币种',
          dataIndex: 'currency',
          scopedSlots: { customRender: 'currency' }
        },
        {
          title: '事项',
          dataIndex: 'matter',
          scopedSlots: { customRender: 'matter' }
        },
        {
          title: '转入/出',
          dataIndex: 'type',
          scopedSlots: { customRender: 'type' }
        },
        {
          title: '金额',
          dataIndex: 'money',
          scopedSlots: { customRender: 'money' }
        },
        {
          title: '状态',
          dataIndex: 'status',
          scopedSlots: { customRender: 'status' }
        },
        {
          title: '操作',
          dataIndex: 'action',
          width: '150px',
          scopedSlots: { customRender: 'action' }
        }
      ],
      // 加载数据方法 必须为 Promise 对象
      loadData: parameter => {
        console.log('loadData.parameter', parameter)
        return getServiceList(Object.assign(parameter, this.queryParam))
          .then(res => {
            return res.result
          })
      },
      selectedRowKeys: [],
      selectedRows: [],

      // custom table alert & rowSelection
      options: {
        alert: { show: true, clear: () => { this.selectedRowKeys = [] } },
        rowSelection: {
          selectedRowKeys: this.selectedRowKeys,
          onChange: this.onSelectChange
        }
      },
      optionAlertShow: true
    }
  },
  computed: {
    // 币种
    currencyMap () {
      return currencyMap
    }
  },
  filters: {
    statusFilter (type) {
      return statusMap[type].text
    },
    statusTypeFilter (type) {
      return statusMap[type].status
    },
    currencyFilter (type) {
      return currencyMap[type].text
    },
    typeFilter (type) {
      return typesMap[type].text
    },
    matterFilter (type) {
      return mattersMap[type].text
    },
    status2Filter (type) {
      return status2Map[type].text
    }
  },
  created () {
    this.tableOption()
    getRoleList({ t: new Date() })
  },
  methods: {
    // ------查询时间选择
    queryTimeChange (date, dateString) {
      console.log(date, dateString)
    },
    tableOption () {
      if (!this.optionAlertShow) {
        this.options = {
          alert: { show: true, clear: () => { this.selectedRowKeys = [] } },
          rowSelection: {
            selectedRowKeys: this.selectedRowKeys,
            onChange: this.onSelectChange,
            getCheckboxProps: record => ({
              props: {
                disabled: record.no === 'No 2', // Column configuration not to be checked
                name: record.no
              }
            })
          }
        }
        this.optionAlertShow = true
      } else {
        this.options = {
          alert: false,
          rowSelection: null
        }
        this.optionAlertShow = false
      }
    },

    handleEdit (record) {
      console.log(record)
      this.$refs.modal.edit(record)
    },
    handleSub (record) {
      if (record.status !== 0) {
        this.$message.info(`${record.no} 订阅成功`)
      } else {
        this.$message.error(`${record.no} 订阅失败，规则已关闭`)
      }
    },
    handleOk () {
      this.$refs.table.refresh()
    },
    onSelectChange (selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    },
    toggleAdvanced () {
      this.advanced = !this.advanced
    },
    resetSearchForm () {
      this.queryParam = {
        date: moment(new Date())
      }
    }
  }
}
</script>
