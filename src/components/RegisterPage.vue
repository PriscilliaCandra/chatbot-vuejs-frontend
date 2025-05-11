<template>
  <div class="auth-container">
    <h2>Register</h2>
    <form @submit.prevent="register">
      <input type="text" v-model="name" placeholder="Nama" required />
      <input type="email" v-model="email" placeholder="Email" required />
      <input
        type="password"
        v-model="password"
        placeholder="Password"
        required
      />
      <input
        type="password"
        v-model="password_confirmation"
        placeholder="Konfirmasi Password"
        required
      />
      <button type="submit">Daftar</button>
      <p class="message" v-if="error">{{ error }}</p>
    </form>
    <router-link to="/login">Sudah punya akun? Login</router-link>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: "",
      email: "",
      password: "",
      password_confirmation: "",
      error: null,
    };
  },
  methods: {
    async register() {
      try {
        await fetch("http://localhost:8000/sanctum/csrf-cookie", {
          credentials: "include",
        });

        const res = await fetch("http://localhost:8000/register", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          credentials: "include",
          body: JSON.stringify({
            name: this.name,
            email: this.email,
            password: this.password,
            password_confirmation: this.password_confirmation,
          }),
        });

        if (!res.ok) {
          const data = await res.json();
          this.error = data.message || "Register gagal.";
          return;
        }

        this.$router.push("/chat");
      } catch (error) {
        this.error = "Terjadi kesalahan. Coba lagi.";
        console.error("Register error:", error);
      }
    },
  },
};
</script>
<style scoped>
.auth-container {
  max-width: 400px;
  margin: 3rem auto;
  padding: 2rem;
  border-radius: 10px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
  background-color: #ffffff;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

h2 {
  margin-bottom: 1.5rem;
  text-align: center;
  color: #333;
}

input {
  width: 80%;
  padding: 0.75rem 1rem;
  margin-bottom: 1rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  transition: border-color 0.3s, box-shadow 0.3s;
  font-size: 1rem;
}

input:focus {
  border-color: #42b983;
  box-shadow: 0 0 0 3px rgba(66, 185, 131, 0.2);
  outline: none;
}

button {
  width: 100%;
  padding: 0.75rem;
  background-color: #42b983;
  color: #fff;
  font-size: 1rem;
  font-weight: bold;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #369f6b;
}

.message {
  color: red;
  margin-top: 0.75rem;
  text-align: center;
}

a {
  display: block;
  margin-top: 1rem;
  text-align: center;
  color: #42b983;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}
</style>
