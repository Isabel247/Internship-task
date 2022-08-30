<template>
  <div class="q-mt-md" v-if="otpSent" style="">
    <p class="text-6">We have sent an OTP Number to your mobile/email.
      <br> Please enter it below and tap Submit:</p>

    <div class="q-pa-sm q-gutter-sm q-ml-md">
      <small class=" text-negative" v-if="otpError">OTP entered does not match!</small>
      <q-input outlined v-model="otpEntered" maxlength="4" style="width: 160px;"
        placeholder="Enter OTP" dense/>
    </div>

    <div class="q-pa-sm q-gutter-md q-ml-sm">
      <q-btn color="primary" style="width: 160px; " @click="checkOtp" label="Submit"/> <br>
      <q-btn v-if="countdown < 1" color="secondary" style="width: 160px;"
       @click="resendOtp" label="Resend"/>
      <p  v-else style="font: 12px;">Resend OTP ... {{ countdown }}s</p>
    </div>
  </div>

  <div class="q-my-md q-gutter-sm" v-else>
    <div class="row q-gutter-xs" style="width: 800px">
      <q-input outlined maxlength="11" v-model="mobile" type="text"
       placeholder="Enter Phone Number" dense/>
      <q-btn color="primary" style="width: 150px;" @click="sendOtp('SMS')"
        label="Send via SMS" :disabled="!mobile"/>
    </div>

    <div class="row q-gutter-xs" style="width: 800px">
      <q-input outlined v-model="email" type="email" placeholder="Enter Email Address" dense />
      <q-btn color="primary" style="width: 150px;" @click="sendOtp('EMAIL')"
       label="Send via Email" :disabled="!email"/>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import { services } from 'src/boot/firebase';

export default ({
  name: 'ConsentOTP',

  props: {
    client: String,
    mdid: String,
    terrid: String,
  },

  setup(props, context) {
    const { CodsService } = services;

    const email = ref();
    const mobile = ref();
    const otpEntered = ref('');
    const otpValue = ref('');
    const otpSent = ref(false);
    const otpError = ref(false);
    const timer = ref(null);
    const countdown = ref(30);

    const startTimer = () => {
      timer.value = setInterval(() => {
        countdown.value -= 1;
      }, 1000);
    };

    const stopTimer = () => {
      clearInterval(this.timer);
      countdown.value = 60;
      startTimer();
    };

    const checkOtp = () => {
      if (otpEntered.value === otpValue.value) {
        context.emit('validOtp', otpEntered.value);
        otpError.value = false;
      } else {
        otpError.value = true;
        context.emit('invalidOtp');
      }
    };

    const sendOtp = async (channelValue) => {
      otpSent.value = true;
      otpError.value = false;
      otpValue.value = (Math.random() * 1000000 + 1000000).toString().substr(0, 4);
      const data = {
        otp: otpValue.value,
        channel: channelValue,
        email: email.value,
        mobile: mobile.value,
      };
      CodsService.sendOtpRequest(
        props.client,
        props.terrid,
        props.mdid,
        data,
      );
      startTimer();
    };

    const resendOtp = async () => {
      CodsService.resendOtpRequest(
        props.client,
        props.terrid,
        props.mdid,
      );
      otpError.value = false;
      otpSent.value = false;
      this.otpEntered = '';
    };

    return {
      email,
      mobile,
      otpSent,
      otpEntered,
      otpError,
      otpValue,
      timer,
      countdown,
      startTimer,
      stopTimer,
      checkOtp,
      sendOtp,
      resendOtp,
    };
  },

});
</script>

<style>
.btn {
  width: 170px;
  height: 40px;
  margin-top: 20px;
}
.otp-container {
  margin-top: 20px;
  display: flex;
}
.resend-otp {
  font: 12px;
  text-align: center;
}
.otp-number {
  width: 200px;
  margin: 0 auto;
}
#otp-submit {
  margin: 15px auto;
  display: block;
  height: 20px;
  width: 140px;
}
</style>
