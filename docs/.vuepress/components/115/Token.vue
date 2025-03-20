<script setup lang="ts">
import { NButton, NSpace, NImage, NAlert, NSpin, NInput } from 'naive-ui';
import { ref } from 'vue';
import { api } from '../api';
const appId = ref('');
const codeVerifier = ref('');
const codeResp = ref({
  uid: '',
  time: 0,
  qrcode: '',
  sign: '',
});
const error1 = ref('');
const qrCode = ref('');
const gettingQrCode = ref(false);
async function generateQrcode() {
  gettingQrCode.value = true;
  error1.value = '';
  const res = await fetch(api(`/alist/115/auth_device_code?app_id=${appId.value}`))
  const data = await res.json()
  gettingQrCode.value = false;
  if (data.error) {
    error1.value = data.error;
    return;
  }
  codeVerifier.value = data.code_verifier;
  codeResp.value = data.resp;
  const url = new URL(api('/qr'))
  url.searchParams.set("text", codeResp.value.qrcode)
  qrCode.value = url.toString();
}
const gettingToken = ref(false);
const error2 = ref('');
const tokenResp = ref({
  access_token: '',
  refresh_token: '',
  expires_in: 0,
});
async function getToken() {
  gettingToken.value = true;
  error2.value = '';
  const res = await fetch(api('/alist/115/get_token'), {
    method: 'POST',
    body: JSON.stringify({
      code_verifier: codeVerifier.value,
      uid: codeResp.value.uid,
    }),
  })
  const data = await res.json()
  gettingToken.value = false;
  if (data.error) {
    error2.value = data.error;
    return;
  }
  tokenResp.value = data.resp;
}
</script>

<template>
  <NSpace vertical size="large">
    <NInput v-model:value="appId" placeholder="APP ID, 不填则使用 Alist 默认提供" />
    <NButton block type="primary" @click="generateQrcode" v-if="!codeVerifier" :loading="gettingQrCode">生成二维码</NButton>
    <NAlert title="Error" type="error" v-if="error1">
      {{ error1 }}
    </NAlert>
    <NSpace v-if="qrCode" vertical>
      <NSpace justify="center">
        <NImage width="300" :src="qrCode" />
      </NSpace>
      <NAlert title="扫描二维码" type="info"
        >使用 115 APP 扫描二维码，然后点击下方按钮</NAlert
      >
      <NButton
        size="large"
        @click="getToken"
        type="info"
        block
        :loading="gettingToken"
        >已扫码</NButton
      >
      <NAlert title="Error" type="error" v-if="error2">
        {{ error2 }}
      </NAlert>
    </NSpace>
    <NSpace vertical v-if="tokenResp.refresh_token">
      <b>refresh_token:</b>
      <NInput
        type="textarea"
        autosize
        readonly
        :value="tokenResp.refresh_token"
      />
      <b>access_token:</b>
      <NInput
        type="textarea"
        autosize
        readonly
        :value="tokenResp.access_token"
      />
    </NSpace>
  </NSpace>
</template>