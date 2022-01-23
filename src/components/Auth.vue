<template>
  <div>
    <section>
      <p>KeyStore:</p>
      <label for="keystore">upload</label>
      <input type="file" id="keystore" accept=".json" @change="keystoreImport" />
      <p>{{ keystoreName || 'Please upload your keystore file.' }}</p>
    </section>

    <label for="password">Password:</label>
    <input type="password" name="password" id="input-password" v-model="password" @keyup.enter="handleLogin" />

    <button @click="handleLogin">Login</button>
  </div>
</template>

<script>
import caver from '../config';

export default {
  data: function () {
    return {
      keystoreData: null,
      keystoreMsg: '',
      keystoreName: '',
      password: '',
      privateKey: '',
    };
  },
  methods: {
    keystoreImport(e) {
      const keystore = e.target.files[0];
      const fileReader = new FileReader();

      fileReader.onload = async (e) => {
        try {
          if (!this.checkValidKeystore(e.target.result)) {
            this.keystoreMsg = 'Invalid keystore file.';
            return;
          }
          this.keystoreName = keystore.name;
          this.keystoreData = e.target.result;
          this.keystoreMsg = 'It is valid keystore. input your password.';
          // console.log('keystoreName', keystore.name);
          // console.log('keystoreData', this.keystoreData);
          // console.log('keystoreMsg', this.keystoreMsg);
          document.querySelector('#input-password').focus();
        } catch (error) {
          console.error(error);
          return;
        }
      };
      fileReader.readAsText(keystore);
    },
    // keystore 파일이 맞는지 확인합니다.
    checkValidKeystore(keystore) {
      const { version, id, address, keyring } = JSON.parse(keystore);

      const isValidKeystore = version && id && address && keyring;
      return isValidKeystore;
    },
    // 지갑 로그인
    handleLogin() {
      const { keystoreData, password } = this;

      try {
        if (!password) {
          return;
        }
        // keystore와 password로 decrypt하여 계정 객체 반환
        const { privateKey } = caver.klay.accounts.decrypt(keystoreData, password);

        // 개인키를 이용한 지갑인스턴스 생성
        const walletInstance = caver.klay.accounts.privateKeyToAccount(privateKey);

        // 계정 객체를 이용해 계정을 인메모리 지갑에 추가
        caver.klay.accounts.wallet.add(walletInstance);

        // 세션스토리지에 계정을 추가함 (브라우저 스토리지에 보관 하는 것은 비 권장되는 방법으로 추후에 수정하도록 할 것.)
        sessionStorage.setItem('walletInstance', JSON.stringify(walletInstance));
      } catch (error) {
        console.error(error);
      }
    },
  },

  updated() {
    console.log(this.password);
  },
};
</script>

<style></style>
