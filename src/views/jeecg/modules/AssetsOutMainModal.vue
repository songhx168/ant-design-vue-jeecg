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
            <a-form-item label="领用后使用公司" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="[ 'assetsCollarCompany', validatorRules.assetsCollarCompany]" placeholder="请输入领用后使用公司"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="领用后使用部门" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="[ 'assetsCollarDepart', validatorRules.assetsCollarDepart]" placeholder="请输入领用后使用部门"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="领用时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择领用时间" v-decorator="[ 'assetsCollarDate', validatorRules.assetsCollarDate]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="领用后使用人" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="[ 'assetsCollarUser', validatorRules.assetsCollarUser]" placeholder="请输入领用后使用人"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="领用备注" :labelCol="labelCol2" :wrapperCol="wrapperCol2">
              <a-textarea v-decorator="['assetsCollarRemark', validatorRules.assetsCollarRemark]" rows="4" placeholder="请输入领用备注"/>
            </a-form-item>
          </a-col>

        </a-row>
      </a-form>

      <!-- 子表单区域 -->
      <a-tabs v-model="activeKey" @change="handleChangeTabs">
        <a-tab-pane tab="资产信息附表" :key="refKeys[0]" :forceRender="true">
          <j-editable-table
            :ref="refKeys[0]"
            :loading="assetsInfoDetailTable.loading"
            :columns="assetsInfoDetailTable.columns"
            :dataSource="assetsInfoDetailTable.dataSource"
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
    name: 'AssetsOutMainModal',
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
          assetsCollarCompany: {rules: [
            {required: true, message: '请输入领用后使用公司!'},
          ]},
          assetsCollarDepart: {rules: [
            {required: true, message: '请输入领用后使用部门!'},
          ]},
          assetsCollarDate: {rules: [
            {required: true, message: '请输入领用时间!'},
          ]},
          assetsCollarUser: {rules: [
            {required: true, message: '请输入领用后使用人!'},
          ]},
          assetsCollarRemark: {rules: [
          ]},
        },
        refKeys: ['assetsInfoDetail', ],
        tableKeys:['assetsInfoDetail', ],
        activeKey: 'assetsInfoDetail',
        // 资产信息附表
        assetsInfoDetailTable: {
          loading: false,
          dataSource: [],
          columns: [
            {
              title: '资产编码',
              key: 'assetCode',
              type: FormTypes.popup,
              popupCode:"assets_out_main",
              destFields:"asset_code,asset_name,company,save_id,asset_type_name,in_date",
              orgFieldse:"asset_code,asset_name,company,id,name,in_date",
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
              validateRules: [{ required: true, message: '${title}不能为空' }],
            },
            {
              title: '资产名称',
              key: 'assetName',
              type: FormTypes.input,
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
              validateRules: [{ required: true, message: '${title}不能为空' }],
            },
            {
              title: '所属公司',
              key: 'company',
              type: FormTypes.input,
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
              validateRules: [{ required: true, message: '${title}不能为空' }],
            },
            {
              title: '资产类别',
              key: 'assetTypeName',
              type: FormTypes.input,
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
              validateRules: [{ required: true, message: '${title}不能为空' }],
            },
            {
              title: '入库时间',
              key: 'inDate',
              type: FormTypes.input,
              width:"200px",
              placeholder: '请输入${title}',
              defaultValue: '',
              validateRules: [{ required: true, message: '${title}不能为空' }],
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
          add: "/assetsoutMain/assetsOutMain/add",
          edit: "/assetsoutMain/assetsOutMain/edit",
          assetsInfoDetail: {
            list: '/assetsoutMain/assetsOutMain/queryAssetsInfoDetailByMainId'
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
        let fieldval = pick(this.model,'assetsCollarCompany','assetsCollarDepart','assetsCollarDate','assetsCollarUser','assetsCollarRemark')
        this.$nextTick(() => {
          this.form.setFieldsValue(fieldval)
        })
        // 加载子表数据
        if (this.model.id) {
          let params = { id: this.model.id }
          this.requestSubTableData(this.url.assetsInfoDetail.list, params, this.assetsInfoDetailTable)
        }
      },
      /** 整理成formData */
      classifyIntoFormData(allValues) {
        let main = Object.assign(this.model, allValues.formValue)

        return {
          ...main, // 展开
          assetsInfoDetailList: allValues.tablesValue[0].values,
        }
      },
      validateError(msg){
        this.$message.error(msg)
      },
     popupCallback(row){
       this.form.setFieldsValue(pick(row,'assetsCollarCompany','assetsCollarDepart','assetsCollarDate','assetsCollarUser','assetsCollarRemark'))
     },

    }
  }
</script>

<style scoped>
</style>