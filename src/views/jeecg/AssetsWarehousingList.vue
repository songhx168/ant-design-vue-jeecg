<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="资产编码">
              <a-input placeholder="请输入资产编码" v-model="queryParam.assetCode"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="资产名称">
              <a-input placeholder="请输入资产名称" v-model="queryParam.assetName"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="入库时间">
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择开始时间" class="query-group-cust" v-model="queryParam.inDate_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择结束时间" class="query-group-cust" v-model="queryParam.inDate_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="资产状态">
                <j-dict-select-tag placeholder="请选择资产状态" v-model="queryParam.assetState" dictCode="asset_state"/>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('资产入库表')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{fixed:true,selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="图片不存在" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="uploadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <assetsWarehousing-modal ref="modalForm" @ok="modalFormOk"></assetsWarehousing-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import AssetsWarehousingModal from './modules/AssetsWarehousingModal'
  import JDictSelectTag from '@/components/dict/JDictSelectTag.vue'
  import JDate from '@/components/jeecg/JDate.vue'
  import { loadCategoryData } from '@/api/api'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'

  export default {
    name: "AssetsWarehousingList",
    mixins:[JeecgListMixin],
    components: {
      JDictSelectTag,
      JDate,
      AssetsWarehousingModal
    },
    data () {
      return {
        description: '资产入库表管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'所属公司',
            align:"center",
            dataIndex: 'company'
          },
          {
            title:'资产编码',
            align:"center",
            dataIndex: 'assetCode'
          },
          {
            title:'资产名称',
            align:"center",
            dataIndex: 'assetName'
          },
          {
            title:'资产类别',
            align:"center",
            dataIndex: 'assetTypeName',
            customRender: (text) => (text ? filterMultiDictText(this.dictOptions['assetTypeName'], text) : '')
          },
          {
            title:'入库时间',
            align:"center",
            dataIndex: 'inDate'
          },
          {
            title:'资产型号',
            align:"center",
            dataIndex: 'assetModel'
          },
          {
            title:'资产来源',
            align:"center",
            dataIndex: 'assetCome_dictText'
          },
          {
            title:'渠道',
            align:"center",
            dataIndex: 'channel'
          },
          {
            title:'金额',
            align:"center",
            dataIndex: 'assetPrice'
          },
          {
            title:'品牌',
            align:"center",
            dataIndex: 'assetBrand'
          },
          {
            title:'购入时间',
            align:"center",
            dataIndex: 'buyDate'
          },
          {
            title:'计量单位',
            align:"center",
            dataIndex: 'assetMeasurement'
          },
          {
            title:'使用期限',
            align:"center",
            dataIndex: 'assetUsefulLife'
          },
          {
            title:'保修起始',
            align:"center",
            dataIndex: 'guaranteeStart'
          },
          {
            title:'过保时间',
            align:"center",
            dataIndex: 'guaranteeEnd'
          },
          {
            title:'存放地',
            align:"center",
            dataIndex: 'saveArea'
          },
          {
            title:'资产状态',
            align:"center",
            dataIndex: 'assetState_dictText'
          },
          {
            title:'使用部门',
            align:"center",
            dataIndex: 'assetsCollarDepart'
          },
          {
            title:'使用人',
            align:"center",
            dataIndex: 'assetsCollarUser'
          },
          {
            title:'领用时间',
            align:"center",
            dataIndex: 'assetsCollarDate'
          },
          {
            title:'清理处理人',
            align:"center",
            dataIndex: 'assetRetirementUser'
          },
          {
            title:'清理原因',
            align:"center",
            dataIndex: 'assetRetirementReason'
          },
          {
            title:'清理时间',
            align:"center",
            dataIndex: 'assetRetirementDate'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/assetswarehousing/assetsWarehousing/list",
          delete: "/assetswarehousing/assetsWarehousing/delete",
          deleteBatch: "/assetswarehousing/assetsWarehousing/deleteBatch",
          exportXlsUrl: "/assetswarehousing/assetsWarehousing/exportXls",
          importExcelUrl: "assetswarehousing/assetsWarehousing/importExcel",
        },
        dictOptions:{},
      }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      initDictConfig(){
        loadCategoryData({code:"B03"}).then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'assetTypeName', res.result)
          }
        })
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>