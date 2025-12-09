<template>
    <ion-page>
      <ion-header>
        <ion-toolbar>
          <ion-title>OTP Verification</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-content fullscreen class="ion-padding">
        <div style="height: 80vh; display: flex; justify-content: center; align-items: center;">
             <div class="w-full max-w-md px-4">
            <ion-card class="w-full p-4 shadow-md rounded-lg otp-card">
                 <!-- <div class="otp-timer">
                     {{ `00:${timer < 10 ? '0' + timer : timer}` }}
                 </div> -->
            <ion-card-content class="flex flex-col items-center">
              <div class="text-gray-50 text-sm mb-4 text-center">Enter OTP</div>
  
              <div class="otp-wrapper">
                <input
                  v-for="(digit, index) in otpDigits"
                  :key="index"
                  ref="otpInputs"
                  type="text"
                  maxlength="1"
                  inputmode="numeric"
                  :value="showOtp ? digit : (digit ? '•' : '')"
                  class="otp-box"
                  @input="handleInput(index, $event)"
                  @keydown.backspace="handleBackspace(index, $event)"
                />
                <ion-icon
                  :icon="otpToggleIcon"
                  class="ml-2 text-xl cursor-pointer"
                  @click="showOtp = !showOtp"
                />
              </div>
  
              <!-- Buttons -->
              <div class="flex flex-col gap-3">
                
                <ion-button expand="block" @click="submitOtp">Submit</ion-button>
                <!-- <ion-button expand="block" fill="outline" @click="resendOtp">Resend OTP</ion-button> -->
              </div>
            </ion-card-content>
          </ion-card>
        </div>
    </div>
      </ion-content>
    </ion-page>
  </template>
  
  <script>
  import {
    IonPage, IonContent, IonCard, IonCardContent,
    IonToolbar, IonTitle, IonHeader, IonButton
  } from '@ionic/vue';
  import axios from 'axios';
  import { alertController } from '@ionic/vue';
  import { IonIcon } from '@ionic/vue';
  import { eye, eyeOff } from 'ionicons/icons';
  import api from '@/api'; // Make sure this path is correct for your api module
  
  export default {
    name: 'Otp',
    components: {
      IonIcon,
      IonPage,
      IonContent,
      IonCard,
      IonCardContent,
      IonToolbar,
      IonTitle,
      IonHeader,
      IonButton
    },
    data() {
      return {
        otpDigits: ['', '', '', ''],
        showOtp: false,
        // timer: 59,
        // intervalId: null,
        // timeoutId: null,
        otpAttempts: 0,
        maxAttempts: 5, 
      };
    },
    computed: {
      otpToggleIcon() {
        return this.showOtp ? eyeOff : eye;
      }
    }, 
    mounted() {
    this.userId = this.loggedInUserId();
    // this.startTimer();
    
  },
    methods: {
      async showAlert(header, message) {
        const alert = await alertController.create({
          header,
          message,
          buttons: ['OK'],
          cssClass: 'custom-alert',
        });
        await alert.present();
      },
  
      handleInput(index, event) {
        const value = event.target.value.replace(/\D/g, '');
        if (value) {
          this.otpDigits[index] = value;
          if (index < 3) {
            this.$refs.otpInputs[index + 1].focus();
          }
        }
      },
  
      handleBackspace(index, event) {
        if (event.key === 'Backspace') {
          if (this.otpDigits[index]) {
            this.otpDigits[index] = '';
          } else if (index > 0) {
            this.$refs.otpInputs[index - 1].focus();
          }
        }
      },
  
      async submitOtp() {
        this.loadderOn();
        const otp = this.otpDigits.join('');

        // if (this.timer === 0) {
        //   await this.showAlert("Expired", "OTP has expired. Please resend to get a new one.");
        //      return;
        // }

        if (otp.length < 4) {
          await this.showAlert("Missing OTP", "Please enter all 4 digits.");
          return;
        }
  
        try {
        const userId = this.loggedInUserId();
        const response = await api.login("/vcp.java/servlet/MobileOTP", {
            email: userId,
            otp: otp,
            type:"CheckOTP",
          });
  
          if (response.data.status === '00') {
            await this.showAlert("Success", "✅ Login Successful");
             this.$router.push('Home');
          } else {
           
            this.otpAttempts++;

            if (this.otpAttempts < this.maxAttempts - 1) {
             await this.showAlert("Error", response.data.message || 'Invalid OTP');
            } else if (this.otpAttempts === this.maxAttempts - 1) {
             await this.showAlert("Warning", "Your 1 attempt is remaining or Your Account May Be Locked ");
            } else if (this.otpAttempts >= this.maxAttempts) {
            //  await this.showAlert("Account Locked", "Your Accouct is Deactivated .Please Contact Your Bank");
            this.clearUserData();
             this.$router.push("login");
             await this.showAlert("Account Locked", "User is Deactivated");
            return;
            }
         }
        } catch (error) {
          console.error(error);
          await this.showAlert("Error", 'Something went wrong while verifying OTP');
        }
        this.loadderOff();
      },
  
      async resendOtp() {
        this.loadderOn();
        this.resetTimer();
        this.otpAttempts = 0;

        try {
            const userId = this.loggedInUserId();

            const response = await api.login("/vcp.java/servlet/MobileOTP", {
            email: userId,
            type: "GenerateOTP", 
          });

         if (response.data.status === '00') {
         await this.showAlert("OTP resent. Please check your messages.");
         } else {
         await this.showAlert("Error", response.data.message || 'Failed to resend OTP');
        }
     } catch (error) {
         console.error(error);
         await this.showAlert("Error", "Something went wrong while resending OTP");
     }
     this.loadderOff();
    } ,
  

    // startTimer() {
    // this.intervalId = setInterval(() => {
    //  if (this.timer > 0) {
    //      this.timer--;
    //  } else {
    //       clearInterval(this.intervalId);
    //     }
    // }, 1000);

    //  this.timeoutId = setTimeout(() => {
    //  }, 60000); 
    // },


    // resetTimer() {
    // clearInterval(this.intervalId);
    // clearTimeout(this.timeoutId);
    // this.timer = 59;

    //  this.startTimer();
    // },
   
    }
  };
  </script>
  
  <style scoped>
  .otp-box {
    width: 30px;
    height: 40px;
    text-align: center;
    font-size: 1.25rem;
    border: 1px solid #ccc;
    border-radius: 6px;
    outline: none;
    background-color: white;
    margin: 0 4px;
  }
  
  .otp-box:focus {
    border-color: #3b82f6;
    box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.3);
  }
  
  .otp-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 8px;
  }
  
  ion-icon {
    font-size: 24px;
    color: #333;
  }

  .otp-card {
  position: relative;
}
.otp-timer {
  position: absolute;
  top: 8px;
  right: 16px;
  font-weight: bold;
  font-size: 14px;
  color: #3880ff;
}

  </style>
  