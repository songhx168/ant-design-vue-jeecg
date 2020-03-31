<template>
  <a-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="所属公司" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'company', validatorRules.company]" placeholder="请输入所属公司"></a-input>
        </a-form-item>
        <a-form-item label="资产编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetCode', validatorRules.assetCode]" placeholder="请输入资产编码"></a-input>
        </a-form-item>
        <a-form-item label="资产名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetName', validatorRules.assetName]" placeholder="请输入资产名称"></a-input>
        </a-form-item>
        <a-form-item label="资产类别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-category-select v-decorator="['assetTypeName', validatorRules.assetTypeName]" pcode="B03" placeholder="请选择资产类别" />
        </a-form-item>
        <a-form-item label="入库时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择入库时间" v-decorator="[ 'inDate', validatorRules.inDate]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="资产型号" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetModel', validatorRules.assetModel]" placeholder="请输入资产型号"></a-input>
        </a-form-item>
        <a-form-item label="资产来源" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag type="list" v-decorator="['assetCome', validatorRules.assetCome]" :trigger-change="true" dictCode="asset_come" placeholder="请选择资产来源"/>
        </a-form-item>
        <a-form-item label="渠道" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'channel', validatorRules.channel]" placeholder="请输入渠道"></a-input>
        </a-form-item>
        <a-form-item label="金额" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'assetPrice', validatorRules.assetPrice]" placeholder="请输入金额" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="品牌" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetBrand', validatorRules.assetBrand]" placeholder="请输入品牌"></a-input>
        </a-form-item>
        <a-form-item label="购入时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择购入时间" v-decorator="[ 'buyDate', validatorRules.buyDate]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="计量单位" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="[ 'assetMeasurement', validatorRules.assetMeasurement]" placeholder="请输入计量单位" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="使用期限" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetUsefulLife', validatorRules.assetUsefulLife]" placeholder="请输入使用期限"></a-input>
        </a-form-item>
        <a-form-item label="保修起始" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择保修起始" v-decorator="[ 'guaranteeStart', validatorRules.guaranteeStart]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="过保时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择过保时间" v-decorator="[ 'guaranteeEnd', validatorRules.guaranteeEnd]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="存放地" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'saveArea', validatorRules.saveArea]" placeholder="请输入存放地"></a-input>
        </a-form-item>
        <a-form-item label="资产状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag type="list" v-decorator="['assetState', validatorRules.assetState]" :trigger-change="true" dictCode="asset_state" placeholder="请选择资产状态"/>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'  
  import JDictSelectTag from "@/components/dict/JDictSelectTag"
  import JCategorySelect from '@/components/jeecg/JCategorySelect'

  export default {
    name: "AssetsWarehousingModal",
    components: { 
      JDate,
      JDictSelectTag,
      JCategorySelect
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
          company: {rules: [
            {required: true, message: '请输入所属公司!'},
          ]},
          assetCode: {rules: [
            {required: true, message: '请输入资产编码!'},
          ]},
          assetName: {rules: [
            {required: true, message: '请输入资产名称!'},
          ]},
          assetTypeName: {rules: [
            {required: true, message: '请输入资产类别!'},
          ]},
          inDate: {rules: [
            {required: true, message: '请输入入库时间!'},
          ]},
          assetModel: {rules: [
          ]},
          assetCome: {rules: [
            {required: true, message: '请输入资产来源!'},
          ]},
          channel: {rules: [
          ]},
          assetPrice: {rules: [
          ]},
          assetBrand: {rules: [
            {required: true, message: '请输入品牌!'},
          ]},
          buyDate: {rules: [
            {required: true, message: '请输入购入时间!'},
          ]},
          assetMeasurement: {rules: [
          ]},
          assetUsefulLife: {rules: [
          ]},
          guaranteeStart: {rules: [
          ]},
          guaranteeEnd: {rules: [
          ]},
          saveArea: {rules: [
            {required: true, message: '请输入存放地!'},
          ]},
          assetState: {rules: [
            {required: true, message: '请输入资产状态!'},
          ]},
        },
        url: {
          add: "/assetswarehousing/assetsWarehousing/add",
          edit: "/assetswarehousing/assetsWarehousing/edit",
        }
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'company','assetCode','assetName','assetTypeName','inDate','assetModel','assetCome','channel','assetPrice','assetBrand','buyDate','assetMeasurement','assetUsefulLife','guaranteeStart','guaranteeEnd','saveArea','assetState','assetsCollarDepart','assetsCollarUser','assetsCollarDate','assetRetirementUser','assetRetirementReason','assetRetirementDate'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
         
        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'company','assetCode','assetName','assetTypeName','inDate','assetModel','assetCome','channel','assetPrice','assetBrand','buyDate','assetMeasurement','assetUsefulLife','guaranteeStart','guaranteeEnd','saveArea','assetState','assetsCollarDepart','assetsCollarUser','assetsCollarDate','assetRetirementUser','assetRetirementReason','assetRetirementDate'))
      },
      handleCategoryChange(value,backObj){
        this.form.setFieldsValue(backObj)
      }

      
    }
  }
</script>