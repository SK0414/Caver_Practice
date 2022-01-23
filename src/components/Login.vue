<template>
  <div class="LoginWrapper">
    <div v-if="!isLogin">
      <div class="loginHeader">
        <h1>Login</h1>
      </div>
      <div class="content" style="margin-top: 30px">
        <div class="inputContainer">
          <p class="inputLabel">KeyStore</p>
          <div class="inputWrapper">
            <p style="display: inline-block; margin-right: 20px; word-break: break-all">{{ keystoreName || 'Please upload your keystore file.' }}</p>
            <label class="inputButton" for="keystore">Upload</label>
            <input class="inputFile" type="file" name="keystore" id="keystore" accept=".json" @change="keystoreImport" />
          </div>
          <p style="font-size: 12px; color: red; margin-top: 2px">{{ keystoreMsg }}</p>
        </div>
        <div class="inputContainer" style="margin-top: 15px">
          <p class="inputLabel">Password</p>
          <div class="inputWrapper">
            <input
              type="password"
              placeholder="Please enter your private password."
              name="password"
              id="inputPassword"
              v-model="password"
              @keyup.enter="handleLogin"
            />
          </div>
          <div class="buttonWrapper">
            <button class="loginBtn" @click="handleLogin">Login</button>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="isLogin">
      <div class="loginHeader">
        <h1>Balance</h1>
      </div>
      <div class="content" style="display: flex; justify-content: center; align-items: center; flex-direction: column; margin-top: 30px">
        <img
          alt="klaytn_logo"
          style="width: 100px"
          src="https://scontent-ssn1-1.xx.fbcdn.net/v/t39.30808-6/271875148_2139642342860940_2578689960750566307_n.jpg?_nc_cat=108&ccb=1-5&_nc_sid=09cbfe&_nc_ohc=plxT5KArxE8AX98CB2N&_nc_ht=scontent-ssn1-1.xx&oh=00_AT9eo8tfOXYoHESsqHDa4JbbVjRXPal0JOh4sxILzGsG2w&oe=61F1EF7D"
        />
        <p style="margin-top: 20px; font-size: 30px">{{ Number(balance).toFixed(4) }} Klay</p>
      </div>
    </div>
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
      balance: null,
      isLogin: sessionStorage.getItem('walletInstance'),
    };
  },
  methods: {
    // state를 초기화합니다.
    resetState() {
      this.keystoreData = null;
      this.keystoreMsg = '';
      this.keystoreName = '';
      this.password = '';
      this.privateKey = '';
    },

    // keystore를 사용자로부터 입력받습니다.
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
          document.querySelector('#inputPassword').focus();
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
    async handleLogin() {
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

        // 세션스토리지에 계정을 추가함 (브라우저 스토리지에 보관 하는 것은 비 권장되는 방법으로 추후에 캡슐화 등으로 수정하도록 할 것.)
        sessionStorage.setItem('walletInstance', JSON.stringify(walletInstance));
        await this.getBalance();
        this.isLogin = true;
        this.resetState();
      } catch (error) {
        console.error(error);
      }
    },

    async getBalance() {
      const address = JSON.parse(sessionStorage.getItem('walletInstance')).address;
      if (!address) return;
      //  주어진 지갑의 peb 단위 현재 잔액 -> 1 Klay 단위 현재 잔액으로 변환
      await caver.klay.getBalance(address).then((balance) => (this.balance = caver.utils.fromPeb(balance, 'KLAY')));
    },
  },
  mounted() {
    if (this.isLogin) this.getBalance();
  },

  updated() {},
};
</script>

<style scoped>
.LoginWrapper {
  margin: auto;
  padding: 10px;
  width: 500px;
  min-height: 300px;
  /* height: 300px; */
  border: 1px solid rgba(0, 0, 0, 0.3);
  border-radius: 30px;
}

.loginHeader {
  display: flex;
  justify-content: center;
  /* border-bottom: 1px solid rgba(0, 0, 0, 0.03); */
}

.loginHeader > h1 {
  font-size: 20px;
  margin: 0;
  padding: 10px;
}

.inputLabel {
  font-size: 14px;
  color: #333333;
}

.inputWrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.inputContainer {
  margin: 0 40px;
}

.inputButton {
  padding: 5px;
  border: 1px solid black;
  border-radius: 30px;
  background: #ffffff;
  cursor: pointer;
}

.inputButton:hover {
  background: rgba(0, 0, 0, 0.03);
}

.inputFile {
  position: absolute;
  clip: rect(0, 0, 0, 0);
}

#inputPassword {
  width: 100%;
  height: 25px;
  margin-top: 10px;
  padding: 1px 15px;
  border-radius: 5px;
}

.buttonWrapper {
  margin-top: 30px;
  text-align: center;
}

.loginBtn {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 54px;
  margin-bottom: 20px;
  border: none;
  border-radius: 27px;
  background-color: #3366ff;
  font-size: 15px;
  color: #ffffff;
  cursor: pointer;
}
</style>
