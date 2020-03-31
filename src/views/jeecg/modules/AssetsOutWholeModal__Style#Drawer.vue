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

        <a-form-item label="领用后使用公司" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetsCollarCompany', validatorRules.assetsCollarCompany]" placeholder="请输入领用后使用公司"></a-input>
        </a-form-item>
        <a-form-item label="领用后使用部门" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetsCollarDepart', validatorRules.assetsCollarDepart]" placeholder="请输入领用后使用部门"></a-input>
        </a-form-item>
        <a-form-item label="领用时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择领用时间" v-decorator="[ 'assetsCollarDate', validatorRules.assetsCollarDate]" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="领用后使用人" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetsCollarUser', validatorRules.assetsCollarUser]" placeholder="请输入领用后使用人"></a-input>
        </a-form-item>
        <a-form-item label="领用备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['assetsCollarRemark', validatorRules.assetsCollarRemark]" rows="4" placeholder="请输入领用备注"/>
        </a-form-item>
        <a-form-item label="资产编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-popup
            v-decorator="['assetcode', validatorRules.assetcode]"
            :trigger-change="true"
            org-fields="asset_code,asset_name,company,id,name,in_date"
            dest-fields="assetcode,assetname,company,saveid,assettypename,indate"
            code="assets_out_main"
            @callback="popupCallback"/>
        </a-form-item>
        <a-form-item label="资产名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'assetname', validatorRules.assetname]" placeholder="请输入资产名称"></a-input>
        </a-form-item>
        <a-form-item label="资产所属公司" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'company', validatorRules.company]" placeholder="请输入资产所属公司"></a-input>
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
    name: "AssetsOutWholeModal",
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
          assetcode: {rules: [
            {required: true, message: '请输入资产编码!'},
          ]},
          assetname: {rules: [
            {required: true, message: '请输入资产名称!'},
          ]},
          company: {rules: [
            {required: true, message: '请输入资产所属公司!'},
          ]},
          assettypename: {rules: [
            {required: true, message: '请输入资产类别!'},
          ]},
          indate: {rules: [
            {required: true, message: '请输入入库时间!'},
          ]},
        },
        url: {
          add: "/assetsoutwhole/assetsOutWhole/add",
          edit: "/assetsoutwhole/assetsOutWhole/edit",
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
          this.form.setFieldsValue(pick(this.model,'assetsCollarCompany','assetsCollarDepart','assetsCollarDate','assetsCollarUser','assetsCollarRemark','assetcode','assetname','company','assettypename','indate'))
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
        this.form.setFieldsValue(pick(row,'assetsCollarCompany','assetsCollarDepart','assetsCollarDate','assetsCollarUser','assetsCollarRemark','assetcode','assetname','company','assettypename','indate'))
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