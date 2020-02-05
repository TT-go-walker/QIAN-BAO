<template>
  <a-modal
    :title="title"
    :width="640"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleSubmit"
    @cancel="handleCancel"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item
          label="商户账号"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
        >
          <a-input v-decorator="['desc', {rules: [{required: true, min: 5, message: '请输入至少五个字符的规则描述！'}]}]" />
        </a-form-item>
        <a-form-item
          label="类型"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
        >
          <a-select defaultValue="传统" style="width: 120px" @change="handleChange">
            <a-select-option value="1">传统</a-select-option>
            <a-select-option value="2">合约</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item
          label="密码"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
        >
          <a-input v-decorator="['password', {rules: [{required: true, min: 6, message: '请输入至少六个字符的密码！'}]}]" />
        </a-form-item>
        <a-form-item
          label="扫币"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
        >
          <a-radio-group @change="handleChange" v-model="value">
            <a-radio :value="1">扫币</a-radio>
            <a-radio :value="2">不扫币</a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item
          label="费率"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
        >
          <a-input v-decorator="['rate', {rules: [{required: true, min: 6, message: '请输入至少六个字符的密码！'}]}]" />
          <a-radio-group @change="handleChange" v-model="value2">
            <a-radio :value="1">实时</a-radio>
            <a-radio :value="2">月结</a-radio>
          </a-radio-group>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
export default {
  data () {
    return {
      labelCol: {
        xs: { span: 24 },
        sm: { span: 7 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 13 }
      },
      visible: false,
      confirmLoading: false,
      value: '',
      value2: '',

      form: this.$form.createForm(this)
    }
  },
  props: {
    title: {
      type: String,
      required: true
    }
  },
  methods: {
    handleChange (e) {
      // console.log('radio checked', e.target.value)
    },
    add () {
      this.visible = true
    },
    handleSubmit () {
      const { form: { validateFields } } = this
      this.confirmLoading = true
      validateFields((errors, values) => {
        if (!errors) {
          console.log('values', values)
          setTimeout(() => {
            this.visible = false
            this.confirmLoading = false
            this.$emit('ok', values)
          }, 1500)
        } else {
          this.confirmLoading = false
        }
      })
    },
    handleCancel () {
      this.visible = false
    }
  }
}
</script>
