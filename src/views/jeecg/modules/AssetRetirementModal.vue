<template>
  <a-modal
    :title="title"
    :width="1200"
    :visible="visible"
    :maskClosable="false"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel">
    <a-spin :spinning="confirmLoading">
      <!-- 主表单区域 -->
      <a-form :form="form">
        <a-row>

          <a-col :span="12">
            <a-form-item label="清理处理人" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="[ 'assetRetirementUser', validatorRules.assetRetirementUser]" placeholder="请输入清理处理人"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="清理时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择清理时间" v-decorator="[ 'assetRetirementDate', validatorRules.assetRetirementDate]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="清理原因" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="[ 'assetRetirementReason', validatorRules.assetRetirementReason]" placeholder="请输入清理原因"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="清理备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="[ 'assetRetirementRemark', validatorRules.assetRetirementRemark]" placeholder="请输入清理备注"></a-input>
            </a-form-item>
          </a-col>

        </a-row>
      </a-form>

      <!-- 子表单区域 -->
      <a-tabs v-model="activeKey" @change="handleChangeTabs">
        <a-tab-pane tab="资产清理明细表" :key="refKeys[0]" :forceRender="true">
          <j-editable-table
            :ref="refKeys[0]"
            :loading="assetRetirementDetailTable.loading"
            :columns="assetRetirementDetailTable.columns"
            :dataSource="assetRetirementDetailTable.dataSource"
            :maxHeight="300"
            :rowNumber="true"
            :rowSelection="true"
            :actionButton="true"/>
        </a-tab-pane>
        
      </a-tabs>

    </a-spin>
  </a-modal>
</template>

<script>

  import pick from 'lodash.pick'
  import { FormTypes,getRefPromise } from '@/utils/JEditableTableUtil'
  import { JEditableTableMixin } from '@/mixins/JEditableTableMixin'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'  

  export default {
    name: 'AssetRetirementModal',
    mixins: [JEditableTableMixin],
    components: {
      JDate,
    },
    data() {
      return {
        labelCol: {
          span: 6
        },
        wrapperCol: {
          span: 16
        },
        labelCol2: {
          span: 3
        },
        wrapperCol2: {
          span: 20
        },
        // 新增时子表默认添加几行空数据
        addDefaultRowNum: 1,
        validatorRules: {
          assetRetirementUser: {rules: [
            {required: true, message: '请输入清理处理人!'},
          ]},
          assetRetirementDate: {rules: [
            {required: true, message: '请输入清理时间!'},
          ]},
          assetRetirementReason: {rules: [
            {required: true, message: '请输入清理原因!'},
          ]},
          assetRetirementRemark: {rules: [
          ]},
        },
        refKeys: ['assetRetirementDetail', ],
        tableKeys:['assetRetirementDetail', ],
        activeKey: 'assetRetirementDetail',
        // 资产清理明细表
        assetRetirementDetailTable: {
          loading: false,
          dataSource: [],
          columns: [
            {
              title: '资产编码',
              key: 'assetCode',
              type: FormTypes.popup,
              popupCode:"asset_retirement",
              destFields:"asset_code,asset_name,company,save_id,asset_type_name,in_date",
              orgFieldse:"asset_code,asset_name,company,id,name,in_date",
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
            },
            {
              title: '资产名称',
              key: 'assetName',
              type: FormTypes.input,
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
            },
            {
              title: '所属公司',
              key: 'company',
              type: FormTypes.input,
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
            },
            {
              title: '资产类别',
              key: 'assetTypeName',
              type: FormTypes.input,
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
            },
            {
              title: '入库时间',
              key: 'inDate',
              type: FormTypes.input,
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
            },

            {
              title: 'asset_code',
              key: 'asset_code',
              type:"hidden"
            },

            {
              title: 'asset_name',
              key: 'asset_name',
              type:"hidden"
            },


            {
              title: 'save_id',
              key: 'save_id',
              type:"hidden"
            },

            {
              title: 'asset_type_name',
              key: 'asset_type_name',
              type:"hidden"
            },

            {
              title: 'in_date',
              key: 'in_date',
              type:"hidden"
            },
          ]
        },
        url: {
          add: "/assetretirement/assetRetirement/add",
          edit: "/assetretirement/assetRetirement/edit",
          assetRetirementDetail: {
            list: '/assetretirement/assetRetirement/queryAssetRetirementDetailByMainId'
          },
        }
      }
    },
    methods: {
      getAllTable() {
        let values = this.tableKeys.map(key => getRefPromise(this, key))
        return Promise.all(values)
      },
      /** 调用完edit()方法之后会自动调用此方法 */
      editAfter() {
        let fieldval = pick(this.model,'assetRetirementUser','assetRetirementDate','assetRetirementReason','assetRetirementRemark')
        this.$nextTick(() => {
          this.form.setFieldsValue(fieldval)
        })
        // 加载子表数据
        if (this.model.id) {
          let params = { id: this.model.id }
          this.requestSubTableData(this.url.assetRetirementDetail.list, params, this.assetRetirementDetailTable)
        }
      },
      /** 整理成formData */
      classifyIntoFormData(allValues) {
        let main = Object.assign(this.model, allValues.formValue)

        return {
          ...main, // 展开
          assetRetirementDetailList: allValues.tablesValue[0].values,
        }
      },
      validateError(msg){
        this.$message.error(msg)
      },
     popupCallback(row){
       this.form.setFieldsValue(pick(row,'assetRetirementUser','assetRetirementDate','assetRetirementReason','assetRetirementRemark'))
     },

    }
  }
</script>

<style scoped>
</style>