<template>
  <ion-page>
    <ion-loading class="custom-loading" :isOpen="showLoader"> </ion-loading>

    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button color="primary"></ion-menu-button>
        </ion-buttons>
        <ion-title>Accounts</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="container">
        <ion-grid class="header-col">
          <ion-row>
            <ion-col>ACCOUNT CODE</ion-col>
            <ion-col class="colname">TYPE</ion-col>
            <ion-col class="balance">BALANCE</ion-col>
          </ion-row>
        </ion-grid>

        <ion-grid v-if="transactions.length" class="data-col">
          <ion-row v-for="(transaction, index) in transactions" :key="index">
            <ion-col>{{ transaction.ACCOUNT }}</ion-col>
            <ion-col class="colname">{{ transaction.TYPE_NAME }}</ion-col>
            <ion-col class="balance">{{ transaction.BALANCE }}</ion-col>
          </ion-row>
        </ion-grid>

        <ion-list v-else>
          <ion-item>
            <ion-label>
              <h3>No transactions were made earlier.</h3>
            </ion-label>
          </ion-item>
        </ion-list>
      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import api from "@/api";
import validator from "validator";

export default {
  watch: {},
  data() {
    return {
      transactions: [],
    };
  },
  computed: {
    isFormValid() {
      return [10, 12].includes(this.email?.length) && this.password?.length > 3;
    },
  },
  mounted() {
    this.fetchTransactions();
  },
  methods: {
    validateForm() {
      this.validation.email = !validator.isEmpty(this.email);
      this.validation.passward = !validator.isEmpty(this.password);
      if (!this.validation.email) {
        return "Pleae enter valid email";
      }
      if (!this.validation.passward) {
        return "Pleae enter valid password";
      }
    },
    async fetchTransactions() {
      try {
        this.loadderOn();
        const userId = this.loggedInUserId();
        const response = await api.post(
          "/vcp.java/servlet/MobileStatement",
          {
            email: userId,
            type: "C",
          }
        );
        // console.log(JSON.stringify(response?.data));

        // console.log("Response:", response.data);

        if (response.data && response.data.statement) {
          // Parse the JSON string into an array of objects
          const statementArray = JSON.parse(response.data.statement);
          if (Array.isArray(statementArray)) {
            this.transactions = statementArray;
          } else {
            console.error("Invalid statement format:", response.data.statement);
          }
        } else {
          console.log("No transactions found."); // Log if no transactions found
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
.header-col {
  font-weight: bold;
  text-align: center;
  background-color: #52545d; /* Light purple header */
  color: white; /* Text color for headers */
}

.balance {
  text-align: right;
}

ion-title {
  /* color: #7c89e3; */
  font-size: 25px;
}
.data-col {
  font-size: 14px;
  text-align: center;
}
.colname {
  /* font-size: 12px; */
  text-align: left;
}
</style>

