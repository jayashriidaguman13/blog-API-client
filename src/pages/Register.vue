<template>
  <div class="home">
    <div class="overlay"></div>
    <div class="content">
      <button class="back-btn" @click="$router.push('/')">Back</button>
      <h1>Register</h1>
      <p>Create your account to access the Astral Archive.</p>
      <div class="form-card">
        <form @submit.prevent="handleRegister" class="auth-form">
          <input type="text" v-model="username" placeholder="Username" class="auth-input" required />
          <input type="email" v-model="email" placeholder="Email" class="auth-input" required />
          <input type="password" v-model="password" placeholder="Password" class="auth-input" required />
          <button type="submit" class="join-btn">Register</button>
        </form>
        <p class="register-link">
          Already have an account? <span @click="$router.push('/login')">Login</span>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import api from "../api.js";
import notyf from "../notyf.js";

export default {
  name: "Register",
  data() {
    return {
      username: "",
      email: "",
      password: ""
    };
  },
  methods: {
    async handleRegister() {
      if (!this.username || !this.email || !this.password) {
        notyf.error("All fields are required.");
        return;
      }
      try {
        const response = await api.post("/users/register", {
          username: this.username,
          email: this.email,
          password: this.password
        });
        notyf.success(response.data.message || "Registration successful");
        setTimeout(() => this.$router.push("/login"), 1500);
      } catch (err) {
        notyf.error(err.response?.data?.message || "Registration failed");
      }
    }
  }
};
</script>

<style scoped>
.home {
  background: url('./src/assets/images/galaxy.jpg') no-repeat center center fixed;
  background-size: cover;
  height: 100vh;
  color: white;
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: 'Quicksand', sans-serif;
  position: relative;
}
.overlay {
  position: fixed;
  top:0; left:0; right:0; bottom:0;
  background-color: rgba(0,0,0,0.4);
  z-index:1;
}
.content {
  z-index:2;
  display:flex;
  flex-direction:column;
  align-items:center;
  max-width: 100vw;
}
.form-card {
  background-color: rgba(0,0,0,0.75);
  padding:2rem;
  border-radius:15px;
  width:100%;
  max-width:400px;
  box-shadow:0 8px 20px rgba(0,0,0,0.5);
}
.auth-input {
  width:100%;
  padding:10px;
  margin-bottom:15px;
  border-radius:5px;
  border:none;
  font-size:1rem;
}
.join-btn {
  background-color:#2a2a72;
  color:white;
  padding:10px 20px;
  font-size:1rem;
  border:none;
  border-radius:5px;
  width:100%;
  cursor:pointer;
}
.back-btn {
  position:absolute;
  top:20px;
  left:20px;
  background-color:#2a2a72;
  color:white;
  padding:10px;
  font-size:1rem;
  border:none;
  border-radius:5px;
  cursor:pointer;
}
.register-link {
  margin-top:15px;
  font-size:0.95rem;
  cursor:pointer;
}
.register-link span {
  color:#7f7faa;
  text-decoration:underline;
}
</style>