<template>
<q-layout>
  <q-page-container class="container q-mt-lg" v-if="!done">
    <div class="row">
      <div class="col">
        <div class="text">
          <div v-if="consentText" v-html="consentText"></div>
          <div v-else>
            <p class="text-justify ">
              By submitting your personal data to
               <span class="text-uppercase"> {{ docsinfo[0] }}</span>
              and affixing your signature, you acknowledge that you have read and understood
              this Form and consent to the collection and processing of your personal data
              as herein provided....
            </p>
            TEXT TO BE PROVIDED BY PRINCIPAL
          </div>
          <p></p>
          <q-separator inset/>
          <div class="row">
            <q-page-container v-if="consentDoNotNeedOtp === 'no'">
              <div class="q-mt-md col-md-6" v-if="!showOtp">
                <p class="header text-weight-bold text-h6">
                  Confirmation via Signature:
                </p>
                <figcaption>{{ consentRequest.mdname }}</figcaption>
                <figcaption>Signature</figcaption>
                <div style="width: 730px">
                  <consent-signature @onSigned="signatureCaptured"/>
                </div>
                <small class="text-negative" v-if="isEmptysign">
                  Please fill-up signautre field
                </small>
                <q-separator class="q-mt-md" inset/>
                <div class="q-my-md">
                  <p class="text-6">
                    You may also request for a One-Time-Password (OTP) <br />if you
                    prefer not to share your signature.
                  </p>
                  <q-btn color="green" label="Use OTP" @click="toggleDiv"/>
                </div>
              </div>

              <div class="q-mt-md col-md-6" v-else>
                <p class="header text-weight-bold text-h6">
                  Confirmation via OTP:
                </p>
                <figcaption>{{ consentRequest.mdname }}</figcaption>
                <q-btn color="green" label="Use Signature" @click="toggleDiv"/>
                <consent-o-t-p
                 :client = "docsinfo[0]"
                 :terrid = "docsinfo[1]"
                 :mdid = "docsinfo[2]"
                 @validOtp="otpCaptured"/>
                <q-separator inset/>
              </div>
            </q-page-container>

            <q-page-container v-else>
              <div class="q-mt-auto">
                <q-btn @click="onAccept" label="I ACCEPT THE ABOVE" />
              </div>
            </q-page-container>
          </div>
          <p>Received by: {{ consentRequest.repname }}</p>
        </div>
      </div>
    </div>
  </q-page-container>

  <q-page-container class="container" v-else>
      <h6 class="text-center">
        Thank you for acknowledging the HCP Consent Form. We look forward to our
        scheduled discussion.
      </h6>
  </q-page-container>
</q-layout>
</template>

<script>
import { ref, watch, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import { ConsentStore } from 'src/stores/consent-store';
import { services } from 'src/boot/firebase';
import { RRLib } from 'rrcoreservice.js';
import ConsentSignature from 'components/ConsentSignature.vue';
import ConsentOTP from 'components/ConsentOTP.vue';

export default ({
  name: 'ConsentForm',

  components: {
    ConsentSignature,
    ConsentOTP,
  },
  setup() {
    const route = useRoute();
    const store = ConsentStore();
    const { CodsService } = services;

    const info = ref();
    const consentRequest = ref();
    const consentInfo = ref();
    const consentForm = ref();
    const consentText = ref('');
    const clientName = ref('');
    const isEmptySign = ref(false);
    const showOtp = ref(false);
    const done = ref(false);
    const consentDoNotNeedOtp = ref('no');

    const toggleDiv = () => {
      showOtp.value = !showOtp.value;
    };

    const docsinfo = ref({
      client: '',
      territoryID: '',
      MDid: '',
    });

    consentRequest.value = store.setConsentRequest(
      docsinfo.value[0],
      docsinfo.value[2],
      docsinfo.value[1],
    );

    onMounted(async () => {
      info.value = await atob(route.params.info).split('/');
      docsinfo.value = info.value;
      consentRequest.value = await store.setConsentRequest(
        docsinfo.value[0],
        docsinfo.value[2],
        docsinfo.value[1],
      );

      consentForm.value = await store.setConsentForm(
        docsinfo.value[0],
        docsinfo.value[2],
      );
      if (consentForm.value) {
        done.value = true;
      }

      consentInfo.value = await store.setConsentInfo(docsinfo.value[0]);
      if (consentInfo.value.consent_text) {
        consentText.value = consentInfo.value.consent_text;
        clientName.value = consentInfo.value.name;
      }
      consentDoNotNeedOtp.value = 'no';
    });

    const otpCaptured = async (otpValue) => {
      done.value = true;
      const detail = {
        mdid: docsinfo.value[2],
        mdname: store.getConsentRequest.mdname,
        territoryId: store.getConsentRequest.terrid,
        repname: store.getConsentRequest.repname,
        mobile: store.getConsentRequest.mobile,
        email: store.getConsentRequest.email,
        otp: otpValue,
        date: RRLib.dateToLongString(new Date()),
      };
      CodsService.saveConsentData(
        docsinfo.value[0],
        docsinfo.value[2],
        docsinfo.value[1],
        detail,
      );
    };

    const signatureCaptured = async (sign) => {
      if (sign !== '') {
        isEmptySign.value = false;
        done.value = true;
        const detail = {
          mdid: docsinfo.value[2],
          mdname: store.getConsentRequest.mdname,
          territoryId: store.getConsentRequest.terrid,
          repname: store.getConsentRequest.repname,
          mobile: store.getConsentRequest.mobile,
          email: store.getConsentRequest.email,
          signature: sign,
          date: RRLib.dateToLongString(new Date()),
        };
        CodsService.saveConsentData(
          docsinfo.value[0],
          docsinfo.value[2],
          docsinfo.value[1],
          detail,
        );
      } else {
        isEmptySign.value = true;
      }
    };

    const onAccept = async () => {
      done.value = true;
      const detail = {
        mdid: docsinfo.value[2],
        mdname: store.getConsentRequest.mdname,
        territoryId: store.getConsentRequest.terrid,
        repname: store.getConsentRequest.repname,
        mobile: store.getConsentRequest.mobile,
        email: store.getConsentRequest.email,
        otp: 'ACCEPT',
        date: RRLib.dateToLongString(new Date()),
      };
      CodsService.saveConsentData(
        docsinfo.value[0],
        docsinfo.value[2],
        docsinfo.value[1],
        detail,
      );
    };

    watch(
      () => route.params.info,
      async (newInfo) => {
        console.log(newInfo);
      },
    );

    return {
      docsinfo,
      consentRequest,
      consentInfo,
      isEmptySign,
      clientName,
      showOtp,
      toggleDiv,
      done,
      consentDoNotNeedOtp,
      consentText,
      otpCaptured,
      signatureCaptured,
      onAccept,
    };
  },
});
</script>

<style>
.container{
  padding: 20px 250px;
}
.svmore-img{
  width: 223px;
  height: 100px;
}
.text{
  margin-bottom: 20px;
  line-height: 1.45;
}
</style>
