<template>
  <ion-page>
    <ion-loading class="custom-loading" :isOpen="showLoader"> </ion-loading>

    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button color="primary"></ion-menu-button>
        </ion-buttons>
        <ion-title>Transfer In Same Bank</ion-title>
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
            <ion-input
              v-model="ben_account"
              labelPlacement="floating"
              value="00.00"
              :class="{
                'ion-invalid': !validation.ben_account,
                'ion-touched': !validation.ben_account,
              }"
              error-text="Invalid beneficiary account"
              @input="handleInput"
            >
              <div slot="label">
                Beneficiary Account Number
                <ion-text color="danger">(Required)</ion-text>
              </div>
            </ion-input>
          </ion-item>

          <ion-item lines="none">
            <!-- <ion-label> Name: {{ beneficiaryName }}</ion-label> -->
            <ion-input
              v-model="beneficiaryName"
              labelPlacement="floating"
              value="00.00"
              placeholder="name"
              :disabled="true"
            >
              <div slot="label">
                Name
                <ion-text color="danger"></ion-text>
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
      reason: "",
      openConfirmationModal: false,
      validation: {
        amount: true,
        ben_account: true,
        reason: false,
      },
      beneficiaryName: "",
      beneficiaries: [],
      utr: "",
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
        validator.isLength(this.ben_account, { min: 14, max: 14 }) &&
        validator.isAlphanumeric(this.ben_account);
      if (!this.validation.amount) {
        return "Pleae enter valid amount to transfer.";
      }
      if (!this.validation.ben_account) {
        return "Pleae enter valid beneficiary account number.";
      }
    },

    async fetchBeneficiaryName() {
      if (this.validation.ben_account && this.ben_account) {
        try {
          this.loadderOn();
          this.ben_account = this.ben_account.trim();
          // console.log("this.ben_account====", this.ben_account);
          const userId = this.loggedInUserId();
          const response = await api.post("/vcp.java/servlet/ShowBeneficiary", {
            email: userId,
            bene_account: this.ben_account,
            type: "S",
          });
          // console.log("Response:", response.data.statement);

          if (response.data && response.data.statement) {
            const BeneArray = JSON.parse(response.data.statement);
            console.log("ResponseBeneArray:", BeneArray);
            if (Array.isArray(BeneArray)) {
              this.beneficiaries = BeneArray;
              // console.log("=====", this.beneficiaries[0].BENIFESARY);
              if (BeneArray.length > 0) {
                this.beneficiaryName = this.beneficiaries[0].BENIFESARY;
              } else {
                this.beneficiaryName = "No Name found";
              }
              if (this.beneficiaryName == ".") {
                this.beneficiaryName = "No Name found";
              }
            } else {
              console.error("Invalid format:", response.data.statement);
              this.beneficiaryName = "Invalid data format";
            }
          } else {
            this.beneficiaryName = "Unable to fetch name.";
          }
        } catch (error) {
          this.beneficiaryName = "Error fetching name. Please try again.";
          console.error(error);
        } finally {
          this.loadderOff();
        }
      } else {
        this.beneficiaryName = "";
      }
    },
    handleInput() {
      this.validateForm();
      this.fetchBeneficiaryName();
    },

    onUserConfirm() {
      this.openConfirmationModal = false;
      this.transfer();
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

        // let random_id = Math.ceil(Math.random() * 1000);
        // // console.log(random_id);
        // var currentDate = new Date();
        // // console.log(currentDate);
        // currentDate.getHours() < 10
        //   ? "0" + currentDate.getHours()
        //   : currentDate.getHours();
        // const datetime =
        //   currentDate.toJSON().slice(0, 10).replace(/-/g, "") +
        //   "" +
        //   (currentDate.getHours() < 10
        //     ? "0" + currentDate.getHours()
        //     : currentDate.getHours()) +
        //   "" +
        //   (currentDate.getMinutes() < 10
        //     ? "0" + currentDate.getMinutes()
        //     : currentDate.getMinutes()) +
        //   "" +
        //   (currentDate.getSeconds() < 10
        //     ? "0" + currentDate.getSeconds()
        //     : currentDate.getSeconds());
        // //(today.getSeconds() < 10 ?  "0" + today.getSeconds() : today.getSeconds())
        // var now_date =
        //   currentDate.getFullYear() +
        //   "-" +
        //   (currentDate.getMonth() + 1) +
        //   "-" +
        //   currentDate.getDate();

        // console.log(datetime + random_id);
        // const utr = datetime + random_id;

        const response = await api.post("/vcp.java/servlet/MobileTrasnaction", {
          email: userId,
          same_bank: "Y",
          bene_account: this.ben_account,
          amount: this.amount,
          reason: this.reason,
          bene_name: this.beneficiaryName,
          utr_number: this.utr,
        });

        console.log(response?.data?.message);
        if (response?.data?.message == "Success") {
          // this.success("Transaction succeed.");
          await this.showAlert("Transaction Succeed.");
          this.openConfirmationModal = false;
          this.loadderOff();
          this.$router.push("Home");
        } else if (
          response?.data?.message == "Failuer" &&
          response?.data?.status == "05"
        ) {
          // this.error("Transaction failed. Insuffucient Account Balance.");
          await this.showAlert("Transaction Failed. Insuffucient Account Balance.");
          this.loadderOff();
        } else {
          // this.error(
          //   "Transaction failed. Please try again or Check Account Code."
          // );
          await this.showAlert("Transaction Failed. Please Try Again or Check Account Code.");
          // this.clearUserData();
          this.loadderOff();
          // this.$router.push("login");
        }
      } catch (error) {
        await this.showAlert("Transaction failed. Please Try Again or Contact to Admin.");
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
  },
};
</script>

<style scoped>
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
