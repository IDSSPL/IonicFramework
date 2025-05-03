<template>
  <ion-page>
    <!-- <ion-loading class="custom-loading" :isOpen="showLoader"> </ion-loading> -->

    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button color="primary"></ion-menu-button>
        </ion-buttons>
        <ion-title>User Confirmation</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="confirmation-dialog">
      <div class="dialog-content" style="overflow-y: auto; max-height: 700px">
        <p>
          I have read and understood the Terms and Conditions applicable to
          Mobile Banking Application Facility provided by Malkapur Urban
          Co-Operative Credit Society Ltd., Malkapur. I confirm to the Society
          that I am the duly Authorized User of the Account. I will be solely
          responsible for protecting any password given by Malkapur Urban
          Co-Operative Credit Society Ltd., Malkapur for the use of the
          facility. Malkapur Urban Co-Operative Credit Society Ltd., Malkapur
          will not be liable for any unauthorized use of any password(s) given
          to me as an Authorized User or for any fraudulent, duplicate or
          erroneous instructions given by use of the password. If I have reason
          to believe that my Mobile Phone Number is / has been allotted to
          another person and / or there has been an unauthorized transaction in
          the Account and / or my Mobile Phone Number is lost, the Authorized
          User shall immediately inform Malkapur Urban Co-Operative Credit
          Society Ltd., Malkapur under acknowledgement about the same. Malkapur
          Urban Co-Operative Credit Society Ltd., Malkapur shall not be
          responsible for any failure to utilize the Facility due to me not
          being within the geographical range within which the Facility is
          offered, Malkapur Urban Co-Operative Credit Society Ltd., Malkapur
          makes no warranty or representation of any kind in relation to the
          system and the network or their function or performance or for any
          loss or damage whenever and howsoever suffered or incurred by me or by
          any person resulting from or in connection with the Facility. Malkapur
          Urban Co-Operative Credit Society Ltd., Malkapur shall under no
          circumstance be held liable if the Facility is not available or there
          is any delay in the carrying out of the instructions for any reasons
          whatsoever including but not limited to natural calamities, legal
          restraints, faults in the telecommunication network or network
          failure, or any other reason beyond the control of Malkapur Urban
          Co-Operative Credit Society Ltd., Malkapur.
        </p>
        <div class="button-container">
          <ion-button @click="login()" color="danger">Disagree</ion-button>
          <ion-button @click="home()" color="success">Agree</ion-button>
        </div>
      </div>
    </div>
    </ion-content>

   
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
    };
  },
  computed: {
    isFormValid() {
      return [10, 12].includes(this.email?.length) && this.password?.length > 3;
    },
  },
  mounted() {
    // this.fetchUserDetails();
  },
  methods: {
    // validateForm() {
    //   this.validation.email = !validator.isEmpty(this.email);
    //   this.validation.passward = !validator.isEmpty(this.password);
    //   if (!this.validation.email) {
    //     return "Pleae enter valid email";
    //   }
    //   if (!this.validation.passward) {
    //     return "Pleae enter valid password";
    //   }
    // },
    async login() {
      try {
        this.loadderOn();
        const userId = this.loggedInUserId();
        // const response = await api.post(
        //   "/vcp.java/servlet/MobileAccountDetails",
        //   {
        //     email: userId,
        //   }
        // );

        // if (response?.data?.name) {
          // this.userDetails = response.data;
          // this.setUserDetails({
          //   email: this.loggedInUserId(),
          //   data: JSON.parse(JSON.stringify(response.data)),
          // });
        // } else {
          // this.error(
          //   "Something went wrong while fetching user details. Please login again."
          // );
           this.clearUserData();
          this.$router.push("login");
        }
       catch (error) {
        this.error(
          "Something went wrong while fetching user details. Please login again."
        );
        this.clearUserData();
        this.$router.push("login");
      }
      this.loadderOff();
    },

    async home() {
      try {
        this.loadderOn();
        const userId = this.loggedInUserId();
        console.log("Response in home:", userId);
        const response = await api.login(
          "/vcp.java/servlet/MobileUserConfirm",
          {
            email: userId,
          }
        );
         console.log("Response:", response.data);
         if (response?.data?.message == "Success") {
          localStorage.setItem("token", this.userId);
      localStorage.setItem("userDetails", JSON.stringify(response.data));
      this.setUserDetails({ email: this.userId, data: response.data });
      this.success("Logged in successfully");
      this.$router.push("Home");
        } else {
          this.clearUserData();
          this.$router.push("login");
        }
        }
       catch (error) {
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

.confirmation-dialog {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.dialog-content {
  background-color: #b6d9c9;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  font-weight: bolder;
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

.container p {
  font-size: 14px;
  line-height: 20px;
  color: #00060b;
  margin: 0;
}

.container h3 {
  font-size: 14px;
  line-height: 22px;
  color: #00060b;
  font-weight: bold;
}

#container a {
  text-decoration: none;
}
</style>
