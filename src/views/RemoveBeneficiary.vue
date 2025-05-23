<template>
  <ion-page>
    <ion-loading class="custom-loading" :isOpen="showLoader"> </ion-loading>

    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button color="primary"></ion-menu-button>
        </ion-buttons>
        <ion-title>Remove Beneficiary</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="container">
        <ion-list>
          <ion-item lines="none">
            <ion-select
              v-model="selectedBeneficiary"
              label="Beneficiary Account Number"
              label-placement="floating"
              fill="outline"
              @ionChange="updateIFSCCode(selectedBeneficiary)"
            >
              <ion-select-option
                v-for="beneacc in beneficiaries"
                :value="beneacc.ACCOUNT_CODE"
                >{{ beneacc.ACCOUNT_CODE }}</ion-select-option
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
              v-model="bene_name"
              labelPlacement="floating"
              value="00.00"
              :disabled="true"
            >
              <div slot="label">
                Beneficiary Name <ion-text color="danger">(Required)</ion-text>
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
        </ion-list>
      </div>
    </ion-content>

    <ion-footer>
      <ion-toolbar>
        <ion-button @click="removebene" expand="full" shape="round"
          >Remove</ion-button
        >
      </ion-toolbar>
    </ion-footer>
  </ion-page>
</template>

<script>
import api from "@/api";
import validator from "validator";

export default {
  watch: {},
  data() {
    return {
      userId: "",
      ben_account: "",
      con_ben_account: "",
      ifsc_code: "",
      bene_name: "",
      bank_name: "",
      selectedBeneficiary: false,
      beneficiaries: [],
      validation: {
        ben_account: false,
        con_ben_account: false,
        ifsc_code: false,
        bene_name: false,
        bank_name: false,
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
    this.fetchBeneficiary();
  },
  methods: {
    validateForm() {
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

      if (!this.validation.ben_account) {
        return "Pleae enter valid beneficiary account number.";
      }
      if (!this.validation.ifsc_code) {
        return "Pleae enter valid IFSC code.";
      }
      if (!this.validation.bank_name) {
        return "Pleae enter valid bank name.";
      }
    },

    updateIFSCCode(selectedAccountCode) {
      const selectedBeneficiary = this.beneficiaries.find(
        (beneacc) => beneacc.ACCOUNT_CODE === selectedAccountCode
      );
      if (selectedBeneficiary) {
        this.ifsc_code = selectedBeneficiary.IFSC_CODE;
        this.bene_name = selectedBeneficiary.BENIFESARY;
        this.bank_name = selectedBeneficiary.BANK_NAME;
        this.ben_account = selectedAccountCode;
        // console.log("ifsc---------" + this.ifsc_code);
        // console.log(this.ben_account);
      } else {
        this.ifsc_code = "";
        this.bank_name = "";
        this.bene_name = "";
        this.ben_account = "";
      }
    },

    async removebene() {
      try {
        const errorMessage = this.validateForm();
        if (errorMessage) {
          this.error(errorMessage);
          return;
        }

        this.loadderOn();
        const userId = this.loggedInUserId();
        const response = await api.post(
          "/vcp.java/servlet/AddBeneficiary",
          {
            email: userId,
            bene_account: this.ben_account,
            bene_ifsc: this.ifsc_code,
            bene_bankname: this.bank_name,
            type: "R",
          }
        );

        if (response?.data?.message == "Success") {
          this.success("Beneficiary Remove Successfully.");
          this.$router.push("Home");
        } else {
          this.error("failed. Please try again or contact to admin.");
          this.clearUserData();
          this.$router.push("login");
        }
      } catch (error) {
        this.error("failed. Please try again or contact to admin.");
        this.clearUserData();
        this.$router.push("login");
      }
      this.loadderOff();
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
