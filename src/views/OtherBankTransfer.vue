<template>
  <ion-page>
    <ion-loading class="custom-loading" :isOpen="showLoader"> </ion-loading>

    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button color="primary"></ion-menu-button>
        </ion-buttons>
        <ion-title>Transfer To Other Bank</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="container">
        <ion-list>
          <ion-item lines="none">
            <ion-input
              v-model="amount"
              labelPlacement="floating"
              value=""
              :class="{
                'ion-invalid': !validation.amount,
                'ion-touched': !validation.amount,
              }"
              error-text="Invalid Amount"
              @input="validateForm"
            >
              <div slot="label">
                Rs <ion-text color="danger">(Required)</ion-text>
              </div>
            </ion-input>
          </ion-item>

          <ion-item lines="none">
            <ion-select
              v-model="selectedBeneficiary"
              label="Beneficiary Account Name"
              label-placement="floating"
              fill="outline"
              @ionChange="updateIFSCCode(selectedBeneficiary)"
            >
              <ion-select-option
                v-for="beneacc in beneficiaries"
                :value="beneacc.BENIFESARY"
                >{{ beneacc.BENIFESARY }}</ion-select-option
              >
            </ion-select>
          </ion-item>
          <ion-item lines="none">
            <ion-input
              v-model="ifsc_code"
              labelPlacement="floating"
              value="00.00"
              :disabled="true"
            >
              <div slot="label">
                IFSC Number <ion-text color="danger">(Required)</ion-text>
              </div>
            </ion-input>
          </ion-item>
          <ion-item lines="none">
            <ion-input
              v-model="ben_account"
              labelPlacement="floating"
              value="00.00"
              :disabled="true"
            >
              <div slot="label">
                Beneficiary Account No.
                <ion-text color="danger">(Required)</ion-text>
              </div>
            </ion-input>
          </ion-item>
          <ion-item lines="none">
            <ion-input
              v-model="bank_name"
              labelPlacement="floating"
              value="00.00"
              :disabled="true"
            >
              <div slot="label">
                Bank Name <ion-text color="danger">(Required)</ion-text>
              </div>
            </ion-input>
          </ion-item>
          <ion-item lines="none">
            <ion-input
              v-model="reason"
              labelPlacement="floating"
              value="00.00"
              placeholder="Reason for transfer (optional)"
              @input="validateForm"
            >
              <div slot="label">
                Reason
                <ion-text color="danger"></ion-text>
              </div>
            </ion-input>
          </ion-item>
          <!-- <ion-item lines="none">
            <ion-label> Unique TXN Number: </ion-label>
            <ion-text>{{ utr }}</ion-text>
          </ion-item> -->
        </ion-list>
      </div>
      <template v-if="openConfirmationModal">
        <ConfirmationBeforeTransaction
          @userConfirmation="onUserConfirm"
        ></ConfirmationBeforeTransaction>
      </template>
    </ion-content>

    <ion-footer>
      <ion-toolbar>
        <ion-button
          @click="openUserConfirmationPopup"
          expand="full"
          shape="round"
          >Transfer</ion-button
        >
      </ion-toolbar>
    </ion-footer>
  </ion-page>
</template>

<script>
import api from "@/api";
import validator from "validator";
import ConfirmationBeforeTransaction from "./ConfirmationBeforeTransaction.vue";
import { alertController } from '@ionic/vue';

export default {
  watch: {},
  components: { ConfirmationBeforeTransaction },
  data() {
    return {
      userId: "",
      amount: "",
      ben_account: "",
      ifsc_code: "",
      bene_name: "",
      bank_name: "",
      reason: "",
      currentFood: "",
      openConfirmationModal: false,
      selectedBeneficiary: false,
      beneficiaries: [],
      utr: "",
      validation: {
        amount: true,
        ben_account: false,
        bene_name: false,
        ifsc_code: false,
        bank_name: false,
        reason: false,
      },
    };
  },
  computed: {
    isFormValid() {
      return [10, 12].includes(this.email?.length) && this.password?.length > 3;
    },
  },
  mounted() {
    this.userId = this.loggedInUserId();
    this.generateUTR();
    this.fetchBeneficiary();
  },
  methods: {
    generateUTR() {
      let random_id = Math.ceil(Math.random() * 100);
      var currentDate = new Date();
      const datetime =
        currentDate.toJSON().slice(0, 10).replace(/-/g, "") +
        "" +
        (currentDate.getHours() < 10
          ? "0" + currentDate.getHours()
          : currentDate.getHours()) +
        "" +
        (currentDate.getMinutes() < 10
          ? "0" + currentDate.getMinutes()
          : currentDate.getMinutes()) +
        "" +
        (currentDate.getSeconds() < 10
          ? "0" + currentDate.getSeconds()
          : currentDate.getSeconds());

      this.utr = datetime + random_id; // Store the generated UTR number
    },

    openUserConfirmationPopup() {
      const errorMessage = this.validateForm();
      if (errorMessage) {
        this.error(errorMessage);
        return;
      }
      this.openConfirmationModal = true;
    },
    validateForm() {
      this.validation.amount = validator.isFloat(this.amount, {
        min: 1.0,
        max: 100000.0,
      });
      this.validation.ben_account =
        !validator.isEmpty(this.ben_account) &&
        // validator.isLength(this.ben_account, { min: 10, max: 14 }) &&
        validator.isAlphanumeric(this.ben_account);
      this.validation.ifsc_code =
        !validator.isEmpty(this.ifsc_code) &&
        validator.isLength(this.ifsc_code, { min: 4, max: 11 });
      this.validation.bank_name =
        !validator.isEmpty(this.bank_name) &&
        validator.isLength(this.bank_name, { min: 4, max: 30 });
      this.validation.bene_name =
        !validator.isEmpty(this.bene_name) &&
        validator.isLength(this.bene_name, { min: 4, max: 30 });

      if (!this.validation.amount) {
        return "Pleae enter valid amount to transfer.";
      }
      if (!this.validation.ben_account) {
        return "Pleae enter valid beneficiary account number.";
      }
      if (!this.validation.ifsc_code) {
        return "Pleae enter valid IFSC code.";
      }
      if (!this.validation.bank_name) {
        return "Pleae enter valid bank name.";
      }
      if (!this.validation.bene_name) {
        return "Pleae enter valid beneficiary name.";
      }
    },
    onUserConfirm() {
      this.transfer();
    },
    // updateIFSCCode(selectedBeneficiary) {
    //   if (selectedBeneficiary) {
    //     this.ifsc_code = selectedBeneficiary.IFSC_CODE;
    //     this.bank_name = selectedBeneficiary.BENIFESARY;
    //     // this.ben_account = selectedBeneficiary.ACCOUNT_CODE;
    //     console.log("ifsc---------" + this.ifsc_code);
    //     console.log(this.ben_account);
    //   } else {
    //     this.ifsc_code = "";
    //     this.bank_name = "";
    //     // this.ben_account = "";
    //   }
    // },
    updateIFSCCode(selectedAccountCode) {
      const selectedBeneficiary = this.beneficiaries.find(
        (beneacc) => beneacc.BENIFESARY === selectedAccountCode
      );
      if (selectedBeneficiary) {
        this.ifsc_code = selectedBeneficiary.IFSC_CODE;
        this.bank_name = selectedBeneficiary.BANK_NAME;
        this.ben_account = selectedBeneficiary.ACCOUNT_CODE;
        this.bene_name = selectedAccountCode;
        // console.log("ifsc---------" + this.ifsc_code);
        // console.log(this.ben_account);
      } else {
        this.ifsc_code = "";
        this.bank_name = "";
        this.bene_name = "";
        this.ben_account = "";
      }
    },
    async transfer() {
      try {
        const errorMessage = this.validateForm();
        if (errorMessage) {
          this.error(errorMessage);
          return;
        }

        this.loadderOn();
        const userId = this.loggedInUserId();
        const response = await api.post(
          "/vcp.java/servlet/MobileTrasnaction",
          {
            email: userId,
            same_bank: "N",
            bene_account: this.ben_account,
            amount: this.amount,
            rtgs_neft: "R",
            bene_ifsc: this.ifsc_code,
            bene_bankname: this.bank_name,
            bene_name: this.bene_name,
            reason: this.reason,
            utr_number: this.utr,
          }
        );

        if (response?.data) {
          this.success("Transaction succeed.");
          this.openConfirmationModal = false;
          this.$router.push("Home");
        } else if (
          response?.data?.message == "Failuer" &&
          response?.data?.status == "05"
        ) {
          await this.showAlert("Transaction failed. Insuffucient Account Balance.");
          this.loadderOff();
        } else {
          await this.showAlert( "Transaction failed. Please try again or contact to admin.");
          this.clearUserData();
          this.$router.push("login");
        }
      } catch (error) {
        await this.showAlert("Transaction failed. Please try again or contact to admin.");
        this.clearUserData();
        this.$router.push("login");
      }
      this.loadderOff();
    },

    async showAlert(header, message) {
     console.log('showAlert called with:', header, message);

     const alert = await alertController.create({
     header : header,
     message: message,
     buttons: ['OK'],
     cssClass: 'custom-alert',
     });
     await alert.present();
    },

    async fetchBeneficiary() {
      try {
        this.loadderOn();
        const userId = this.loggedInUserId();
        const response = await api.post(
          "/vcp.java/servlet/ShowBeneficiary",
          {
            email: userId,
            type: "B",
          }
        );

        // console.log(JSON.stringify(response?.data));
        // console.log("Response:", response.data);

        if (response.data && response.data.statement) {
          // Parse the JSON string into an array of objects
          const BeneArray = JSON.parse(response.data.statement);
          if (Array.isArray(BeneArray)) {
            this.beneficiaries = BeneArray;
          } else {
            console.error("Invalid format:", response.data.statement);
          }
        } else {
          console.log("No transactions found.");
        }
      } catch (error) {
        this.error("Something went wrong while fetching transaction details.");
        this.clearUserData();
        this.$router.push("login");
      }
      this.loadderOff();
    },
  },
};
</script>

<style scoped>
/* Card-like form appearance */
ion-list {
  background: var(--card-background);
  border-radius: 16px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  padding: 20px;
  margin-bottom: 20px;
}
#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
