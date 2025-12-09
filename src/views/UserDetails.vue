<template>
  <ion-page>
    <ion-loading class="custom-loading" :isOpen="showLoader"> </ion-loading>

    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button color="primary"></ion-menu-button>
        </ion-buttons>
        <ion-title>User Details</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="container">
        <ion-list>
          <!-- <ion-item lines="none">
            <ion-input
              v-model="email"
              labelPlacement="floating"
              value="hi@ionic.io"
              :disabled="true"
             
              error-text="Invalid email"
              @input="validateForm"
            >
              <div slot="label">
                Email <ion-text color="danger">(Required)</ion-text>
              </div>
            </ion-input>
          </ion-item> -->

          <!-- <ion-item lines="none">
            <ion-input
              v-model="deviceid"
              labelPlacement="floating"
              value="hi@ionic.io"
              :disabled="true"
             
              error-text="Invalid deviceid"
              @input="validateForm"
            >
              <div slot="label">
                Device id <ion-text color="danger">(Required)</ion-text>
              </div>
            </ion-input>
          </ion-item> -->
       
          <ion-item>
            <ion-label>
              <h3 class="h3">Name</h3>
              <p>{{ userDetails.name }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3 class="h3">Balance</h3>
              <p>Rs {{ userDetails.balance }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3 class="h3">Email</h3>
              <p>{{ userDetails.email_id || "-" }}</p>
            </ion-label>
          </ion-item>

          <!-- <ion-item>
            <ion-label>
              <h3 class="h3">device Id</h3>
              <p>{{ userDetails.phonemobile || "-" }}</p>
            </ion-label>
          </ion-item> -->

          <ion-item>
            <ion-label>
              <h3 class="h3">Address</h3>
              <p v-if="userDetails.address1">{{ userDetails.address1 }}</p>
              <p v-if="userDetails.address2">{{ userDetails.address2 }}</p>
              <p v-if="userDetails.address3">{{ userDetails.address3 }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3 class="h3">City</h3>
              <p>{{ userDetails.city || "-" }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-label>
              <h3 class="h3">Pin Code</h3>
              <p>{{ userDetails.zip || "-" }}</p>
            </ion-label>
          </ion-item>
        </ion-list>
      </div>
    </ion-content>

    <!-- <ion-footer>
      <ion-toolbar>
        <ion-button @click="login" expand="full" shape="round"
          >Login</ion-button
        >
      </ion-toolbar>
    </ion-footer> -->
  </ion-page>
</template>

<script>
import api from "@/api";
import validator from "validator";

export default {
  props: {},
  watch: {},
  data() {
    return {
      userDetails: {},
      email:"",
      deviceid: "",
    };
  },
  computed: {
    isFormValid() {
      return [10, 12].includes(this.email?.length) && this.password?.length > 3;
    },
  },
  mounted() {
    this.fetchUserDetails();
    this.email = this.loggedInUserId();
    this.deviceid = this.loggedInDeviceId();
  },
  methods: {
    validateForm() {
      this.validation.email = !validator.isEmpty(this.email);
      this.validation.passward = !validator.isEmpty(this.password);
      if (!this.validation.email) {
        return "Please enter valid email";
      }
      if (!this.validation.passward) {
        return "Please enter valid password";
      }
    },
    async fetchUserDetails() {
      try {
        this.loadderOn();
        const userId = this.loggedInUserId();
        const response = await api.post(
          "/vcp.java/servlet/MobileAccountDetails",
          {
            email: userId,
          }
        );

        if (response?.data?.name) {
          this.userDetails = response.data;
          this.setUserDetails({
            email: this.loggedInUserId(),
            data: JSON.parse(JSON.stringify(response.data)),
          });
        } else {
          this.error(
            "Something went wrong while fetching user details. Please login again."
          );
          this.clearUserData();
          this.$router.push("login");
        }
      } catch (error) {
        this.error(
          "Something went wrong while fetching user details. Please login again."
        );
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

.h3 {
  font-size: 16px;
  font-weight: bold;
}

.container p {
  font-size: 14px;
  line-height: 22px;
  color: teal;

  font-weight: bold;
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
