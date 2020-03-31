<template>
  <a-drawer
    :title="title"
    :width="width"
    placement="right"
    :closable="false"
    @close="close"
    :visible="visible">
  
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="清理处理人" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetRetirementUser', validatorRules.assetRetirementUser]" placeholder="请输入清理处理人"></a-input>
        </a-form-item>
        <a-form-item label="清理时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择清理时间" v-decorator="[ 'assetRetirementDate', validatorRules.assetRetirementDate]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="清理原因" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetRetirementReason', validatorRules.assetRetirementReason]" placeholder="请输入清理原因"></a-input>
        </a-form-item>
        <a-form-item label="清理备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['assetRetirementRemark', validatorRules.assetRetirementRemark]" rows="4" placeholder="请输入清理备注"/>
        </a-form-item>
        <a-form-item label="资产编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-popup
            v-decorator="['assetcode', validatorRules.assetcode]"
            :trigger-change="true"
            org-fields="asset_code,asset_name,company,id,name,in_date"
            dest-fields="assetcode,assetname,company,saveid,assettypename,indate"
            code="asset_retirement"
            @callback="popupCallback"/>
        </a-form-item>
        <a-form-item label="资产名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetname', validatorRules.assetname]" placeholder="请输入资产名称"></a-input>
        </a-form-item>
        <a-form-item label="资产所属公司" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'company', validatorRules.company]" placeholder="请输入资产所属公司"></a-input>
        </a-form-item>
        <a-form-item label="保存的资产类编编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'saveid', validatorRules.saveid]" placeholder="请输入保存的资产类编编码"></a-input>
        </a-form-item>
        <a-form-item label="资产类别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assettypename', validatorRules.assettypename]" placeholder="请输入资产类别"></a-input>
        </a-form-item>
        <a-form-item label="入库时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择入库时间" v-decorator="[ 'indate', validatorRules.indate]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>
        
      </a-form>
    </a-spin>
    <a-button type="primary" @click="handleOk">确定</a-button>
    <a-button type="primary" @click="handleCancel">取消</a-button>
  </a-drawer>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'  
  
  export default {
    name: "AssetRetirementNewModal",
    components: { 
      JDate,
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
          assetcode: {rules: [
            {required: true, message: '请输入资产编码!'},
          ]},
          assetname: {rules: [
            {required: true, message: '请输入资产名称!'},
          ]},
          company: {rules: [
            {required: true, message: '请输入资产所属公司!'},
          ]},
          saveid: {rules: [
            {required: true, message: '请输入保存的资产类编编码!'},
          ]},
          assettypename: {rules: [
            {required: true, message: '请输入资产类别!'},
          ]},
          indate: {rules: [
            {required: true, message: '请输入入库时间!'},
          ]},
        },
        url: {
          add: "/assetretirementnew/assetRetirementNew/add",
          edit: "/assetretirementnew/assetRetirementNew/edit",
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
          this.form.setFieldsValue(pick(this.model,'assetRetirementUser','assetRetirementDate','assetRetirementReason','assetRetirementRemark','assetcode','assetname','company','saveid','assettypename','indate'))
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
        this.form.setFieldsValue(pick(row,'assetRetirementUser','assetRetirementDate','assetRetirementReason','assetRetirementRemark','assetcode','assetname','company','saveid','assettypename','indate'))
      }
      
    }
  }
</script>

<style lang="less" scoped>
/** Button按钮间距 */
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    float: right;
  }
</style>