/**
* @Author: limeng
* @Date: 2019-08-18 21:40
* @Description:
*/
<template>
  <div class="column-view">
    <van-nav-bar
      title="任务币提现"
      left-text="返回"
      fixed
      left-arrow
      @click-left="onNavBarLeftPressed">
    </van-nav-bar>
    <div class="under-nav flex1 height0 recharge">
      <van-cell-group>
        <van-cell title="合作商级别" :value="partnerName" :label="partnerDescription"/>
        <van-cell title="账户余额" :value="missionCoin"/>
        <van-field label="提现金额" v-model="amount" placeholder="请输入提现金额" :error-message="feeDescription"
                   class="withdrawal-mission-coin">
          <span slot="right-icon">元</span>
        </van-field>
        <van-cell title="提现方式" value="支付宝" is-link/>
      </van-cell-group>
      <button @click="onRechargePressed">提&nbsp;&nbsp;&nbsp;现</button>
      <p>提示：
        1、提现时间为工作日的早9:00~17:00
        2、提现需要审核，审核时间约为1个工作日
        3、不同级别合作商，提现手续费不同
      </p>
    </div>
  </div>
</template>

<script>
  import { WITHDRAWAL_MISSION_COIN_URL } from "@/config/host"
  import { Dialog } from "vant"

  export default {
    name: "WithdrawalMissionCoin",
    data () {
      return {
        amount: null
      }
    },
    watch: {
      amount (newValue, oldValue) {
        this.amount = newValue.replace(/[^0-9]/ig, "")
      }
    },
    mounted () {

    },
    computed: {
      partnerName () {
        return this.$store.getters.partner.name
      },
      partnerDescription () {
        return `提现手续费：${this.$store.getters.partner.missionPaymentPercent}%`
      },
      missionCoin () {
        return this.$store.getters.user.missionCoin / 100.0 + "元"
      },
      feeDescription () {
        if (!this.amount || this.amount.length === 0) {
          return null
        }
        const number = parseInt(this.amount)
        const fee = number * this.$store.getters.partner.missionPaymentPercent / 100.0
        if ((number + fee) * 100 > this.$store.getters.user.missionCoin) {
          return "账户余额不足"
        }
        return `手续费：${fee}元，总计扣除：${(number + fee)}元`
      }
    },
    methods: {
      onNavBarLeftPressed () {
        this.$router.back()
      },
      async onRechargePressed () {
        if (!this.amount || parseInt(this.amount) < 1) {
          this.toastShortMsg("请输入正确的提现金额")
          return
        }
        const amount = parseInt(this.amount)
        const fee = amount * this.$store.getters.partner.missionPaymentPercent / 100.0
        if ((amount + fee) * 100 > this.$store.getters.user.missionCoin) {
          this.toastShortMsg("账户余额不足")
          return
        }
        if (!this.$store.getters.user.alipay) {
          Dialog.confirm({
            message: "提现需绑定支付宝账号",
            confirmButtonText: "去绑定"
          }).then(async () => {
            this.$router.push({ name: "AlipayAccount" })
          })
          return
        }
        let number = parseInt(this.amount) * 100
        const result = await this.httpPostWithLoading(WITHDRAWAL_MISSION_COIN_URL, {
          amount: number
        })
        if (result.success) {
          this.toastMsg("提交成功，申请将在1个工作日审核完成")
          this.$store.commit("updateUser", result.data)
        }
      }
    }
  }
</script>

<style lang="less">
  .withdrawal-mission-coin {
    .van-field__error-message {
      color: #969799
    }
  }
</style>

<style scoped lang="less">
  .recharge {
    button {
      width: 300px;
      height: 36px;
      line-height: normal;
      border: solid 1px #81d4cb;
      border-radius: 4px;
      color: white;
      font-size: 16px;
      background-color: #81d4cb;
      text-align: center;
      margin: 20px auto;
      display: block;
    }

    .risk {
      margin-top: 20px;
      color: #ff4444;
    }

    p {
      white-space: pre-wrap;
      margin-left: 10px;
      margin-right: 10px;
      color: #969799;
    }
  }
</style>
