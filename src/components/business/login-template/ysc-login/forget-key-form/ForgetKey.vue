<!-- 忘记密码 -->
<template>
  <div>
    <Steps :current="current"
           size="small">
      <Step title="确认账号"></Step>
      <Step title="重置密码"></Step>
      <Step title="找回完成"></Step>
    </Steps>
    <div v-if="current === 0"
         style="margin-top: 48px">
      <Form ref="forgetRef"
            :model="forgetForm"
            :rules="forgetRule">
        <FormItem prop="supplierPhone">
          <Input v-model.trim="forgetForm.supplierPhone"
                 placeholder="请输入手机号">
          </Input>
        </FormItem>
        <div>
          <!-- <TCaptcha ref="TCaptcha"></TCaptcha> -->
        </div>
        <div class="next-btn">
          <Button type="primary"
                  :loading="oneLoading"
                  @click="nextOne">下一步</Button>
          <Button class="cancel"
                  @click="backClick">取消</Button>
        </div>
      </Form>
    </div>
    <div v-if="current === 1"
         style="margin-top: 48px">
      <Form ref="forgetRef"
            :model="forgetForm"
            :rules="forgetRule">
        <FormItem v-show="false">
          <Input type="password"
                 style="display: none">
          </Input>
        </FormItem>
        <FormItem prop="messageCode">
          <Input v-model.trim="forgetForm.messageCode"
                 placeholder="请输入 6 位短信验证码">
          </Input>
        </FormItem>
        <FormItem prop="loginKey">
          <Input v-model.trim="forgetForm.loginKey"
                 type="password"
                 placeholder="请输入密码">
          </Input>
        </FormItem>
        <FormItem prop="newLoginKey">
          <Input v-model.trim="forgetForm.newLoginKey"
                 type="password"
                 placeholder="请确认密码">
          </Input>
        </FormItem>
        <div class="next-btn">
          <Button type="primary"
                  :loading="twoLoading"
                  @click="nextTwo">下一步</Button>
          <Button class="cancel"
                  @click="backClick">取消</Button>
        </div>
      </Form>
    </div>
    <div v-if="current === 2"
         style="margin-top: 64px;text-align: center">
      <Icon type="md-checkmark-circle"
            size="80"
            color="#19be6b" />
      <h2>重置成功</h2>
      <div style="margin-top: 48px">
        <span class="span-btn">{{timeCount}}</span>
        <span>秒后</span>
        <span class="span-btn"
              @click="backClick">返回登录</span>
      </div>
    </div>
  </div>
</template>

<script>
import {
  postMessageCode,
  postForgetKey
} from '@/api/login'
// import TCaptcha from '@/components/public/t-captcha'
export default {
  // components: { TCaptcha },
  data () {
    const validateAgainNewKey = (rule, value, callback) => {
      if (this.forgetForm.newLoginKey === '') {
        callback(new Error('请再次输入新密码'))
      } else if (this.forgetForm.newLoginKey !== this.forgetForm.loginKey) {
        callback(new Error('两次输入不一致'))
      } else {
        callback()
      }
    }
    return {
      current: 0,
      oneLoading: false,
      twoLoading: false,
      timeCount: 5, // 倒计时 5秒
      forgetForm: {
        supplierPhone: '',
        messageCode: '',
        loginKey: '',
        newLoginKey: '',
        ticket: '',
        randStr: ''
      },
      // 表单验证
      forgetRule: {
        supplierPhone: [{ required: true, type: 'string', message: '手机号不能为空', trigger: 'change' }],
        messageCode: [
          { required: true, message: '短信验证码不能为空', trigger: 'change' },
          {
            type: 'string',
            len: 6,
            message: '请输入 6 位短信验证码',
            trigger: 'change'
          }
        ],
        loginKey: [
          { required: true, message: '密码不能为空', trigger: 'change' },
          { message: '密码必须为数字和字母组合(6 ~ 20)', pattern: /^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z_]{6,20}$/ }
        ],
        newLoginKey: [
          { required: true, validator: validateAgainNewKey, trigger: 'change' }
        ]
      }
    }
  },
  computed: {},
  watch: {
    current () {
      if (this.current === 2) {
        this.findSuccessCount()
      }
    }
  },

  // 生命周期 - 挂载完成（可以访问 DOM 元素）
  mounted () {
  },
  methods: {
    // 获取短信验证码
    async getMessageCode () {
      this.oneLoading = true
      const { code, message } = await postMessageCode(this.forgetForm)
      if (code === 10000) {
        this.oneLoading = false
        this.$Message.success(message)
        this.next()
      } else {
        this.oneLoading = false
      }
    },

    // 重置密码
    async forgetKey () {
      this.twoLoading = true
      const { code, message } = await postForgetKey(this.forgetForm)
      if (code === 10000) {
        this.next()
        this.$Message.success(message)
        this.twoLoading = false
      } else {
        this.twoLoading = false
      }
    },
    // 确认账号下一步
    nextOne () {
      this.$refs['forgetRef'].validate((valid) => {
        if (valid) {
          this.$refs['TCaptcha'].createCaptcha((data) => {
            this.forgetForm.ticket = data.ticket
            this.forgetForm.randStr = data.randStr
            this.getMessageCode()
          })
        }
      })
    },

    // 重置密码下一步
    nextTwo () {
      this.$refs['forgetRef'].validate((valid) => {
        if (valid) {
          this.forgetKey()
        }
      })
    },

    next () {
      this.current += 1
    },
    // 上传失败读秒倒计时
    findSuccessCount () {
      const timer = setInterval(() => {
        if (this.timeCount > 0) {
          this.timeCount--
        } else if (this.timeCount === 0) {
          clearInterval(timer)
          this.backClick()
        }
      }, 1000)
    },
    backClick () {
      this.$emit('transmitHideforgetKey')
    }
  }
}
</script>
<style scoped lang='less'>
@import "./forget-key.less";
</style>
