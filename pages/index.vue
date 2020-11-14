<template>
  <div class="container">
    <div class="contact-form">
      <h2>Contact Us</h2>

      <form @submit.prevent="onSubmit">
        <input
          autocomplete="off"
          placeholder="Email"
          type="email"
          v-model="email"
        />

        <input
          autocomplete="off"
          placeholder="Phone number"
          type="text"
          v-model="phoneNumber"
        />

        <textarea
          autocomplete="off"
          placeholder="Enter your message"
          type="text"
          v-model="message"
          rows="4"
        />

        <button type="submit" v-if="!loading">Send Message</button>

        <p v-else>Loading...</p>
      </form>

      <div class="message" v-if="notificationMessage">
        <p class="notification-text" :style="errorStatus ? 'color: red;' : ''">
          {{ notificationMessage }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  async mounted() {
    // Initialize recaptcha instance
    try {
      await this.$recaptcha.init();
    } catch (e) {
      console.log(e);
    }
  },
  data() {
    return {
      email: "",
      phoneNumber: "",
      message: "",
      notificationMessage: "",
      errorStatus: false,
      loading: false
    };
  },
  watch: {
    notificationMessage(newValue) {
      // check if the new notification message is not of a truthy value
      if (!Boolean(newValue)) return;

      // Clear notification message after 5 seconds
      const context = this;
      setTimeout(() => {
        context.notificationMessage = "";
      }, 5000);
    }
  },
  methods: {
    async onSubmit() {
      try {
        this.loading = true;

        // Start the verification process
        const response = await this.verifyCaptcha();

        // Display error message if verification was not successful
        if (!response.success) {
          this.$recaptcha.reset();
          this.loading = false;
          this.errorStatus = true;
          this.notificationMessage =
            "There was an error with your reCaptcha verification. Please try again.";
          return;
        }

        // If verification was successful, send the message
        await this.sendMessage();

        this.errorStatus = false;
        this.notificationMessage =
          "Thank you for reaching out. We will get back to you as soon as possible";

        this.loading = false;
        this.$recaptcha.reset();
      } catch (error) {
        this.loading = false;
        console.log(error);
      }
    },
    async verifyCaptcha() {
      try {
        const token = await this.$recaptcha.execute();

        const response = await this.$axios.$post(
          `/captcha-api/siteverify?secret=${process.env.SECRET_KEY}&response=${token}`
        );

        return response;
      } catch (error) {
        this.loading = false;
        return error;
      }
    },
    async sendMessage() {
      // The send message logic goes in here
    }
  }
};
</script>

<style scoped>
.container {
  margin: 0;
  padding: 0;
  height: 100vh;
  width: 100vw;
  display: flex;
  align-items: center;
  background-color: whitesmoke;
}

.contact-form {
  background-color: white;
  width: 30%;
  margin: 0 auto;
  box-shadow: 1px 2px 6px grey;
  padding: 15px;
}

h2 {
  text-align: center;
  padding-bottom: 10px;
}

input,
textarea {
  width: 100%;
  margin-bottom: 10px;
}

input,
button {
  height: 30px;
}

button {
  text-align: center;
  width: 40%;
  background-color: green;
  color: white;
  border: none;
  margin-bottom: 10px;
}

button:hover {
  cursor: pointer;
  width: 41%;
  height: 31px;
}

.message {
  border: solid 1px grey;
  min-height: 30px;
  display: flex;
  align-items: center;
  padding: 5px;
}

.notification-text {
  font-size: 14px;
}
</style>
