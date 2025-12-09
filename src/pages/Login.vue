<template>
  <div class="home">
    <div class="overlay"></div>
    <div class="content">
      <button class="back-btn" @click="$router.push('/')">Back</button>
      <h1>Login</h1>
      <p>Enter your credentials to access the site.</p>
      <div class="login-card">
        <form @submit.prevent="handleLogin" class="login-form">
          <input
            type="email"
            v-model="email"
            placeholder="Email"
            class="login-input"
            required
          />
          <input
            type="password"
            v-model="password"
            placeholder="Password"
            class="login-input"
            required
          />
          <button type="submit" class="join-btn">
            {{ isLoggedIn ? "Logout" : "Login" }}
          </button>
        </form>
        <p class="register-link">
          Don't have an account?
          <span @click="$router.push('/register')">Register</span>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import api from "../api.js";
import notyf from "../notyf.js";

export default {
  name: "Login",
  data() {
    return {
      email: "",
      password: "",
      isLoggedIn: !!localStorage.getItem("token")
    };
  },
  methods: {
    async handleLogin() {
      if (this.isLoggedIn) return this.handleLogout();
      try {
        const response = await api.post("/users/login", {
          email: this.email,
          password: this.password
        });
        localStorage.setItem("token", response.data.token);
        this.isLoggedIn = true;
        notyf.success(response.data.message || "Login successful");
        setTimeout(() => this.$router.push("/"), 1000);
      } catch (err) {
        notyf.error(err.response?.data?.message || "Login failed");
      }
    },
    handleLogout() {
      localStorage.removeItem("token");
      this.isLoggedIn = false;
      notyf.success("Logged out successfully");
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
.login-card {
  background-color: rgba(0,0,0,0.75);
  padding:2rem;
  border-radius:15px;
  width:100%;
  max-width:400px;
  box-shadow:0 8px 20px rgba(0,0,0,0.5);
}
.login-input {
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