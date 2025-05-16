<!-- hilangin scroll di luar -->

<!-- munculin tanggalnya dan history -->

<template>
  <div class="chat-wrapper">
    <div class="header-bar">
      <button @click="logoutUser" class="logout-button">Logout</button>
    </div>
    <div class="chat-window" ref="chatWindow">
      <div
        v-for="(msg, index) in messages"
        :key="index"
        :class="['message', msg.sender]"
      >
        <div class="bubble">
          {{ msg.text }}
          <div v-if="msg.created_at" class="timestamp">{{ msg.created_at }}</div>
        </div>
      </div>
      <div v-if="loading" class="message bot">
        <div class="bubble loading">Typing...</div>
      </div>
    </div>

<div class="input-area">
  <input
    type="text"
    v-model="input"
    @keyup.enter="sendMessage"
    placeholder="Chat here..."
  />
  <button @click="sendMessage" :disabled="loading || !input.trim()">
    Send
  </button>
</div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      input: "",
      messages: [],
      loading: false,
    };
  },
  methods: {
    async sendMessage() {
      const text = this.input.trim();
      if (!text) return;

      this.messages.push({ sender: "user", text });
      this.input = "";
      this.loading = true;

      try {
        const token = localStorage.getItem("token");
        if (!token) {
          throw new Error("No authentication token found");
        }

        const res = await fetch("http://localhost:8000/api/chat", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json",
            "Authorization": `Bearer ${token}`
          },
          credentials: "include",
          body: JSON.stringify({ text }),
        });

        if (!res.ok) {
          if (res.status === 401) {
            // Token tidak valid atau expired
            localStorage.removeItem("token");
            this.$router.push("/login");
            return;
          }
          throw new Error("Failed to fetch: " + res.status);
        }

        const data = await res.json();
        this.messages.push({
          sender: "bot",
          text: data.result || "Gagal mendapatkan respons",
        });
      } catch (error) {
        console.error("Error during request:", error);
        this.messages.push({
          sender: "bot",
          text: "Terjadi kesalahan: " + error.message
        });
      } finally {
        this.loading = false;
        this.$nextTick(() => {
          this.scrollToBottom();
        });
      }
    },

    scrollToBottom() {
      const container = this.$refs.chatWindow;
      container.scrollTop = container.scrollHeight;
    },

    async fetchChatHistory() {
      try {
        const token = localStorage.getItem("token");
        if (!token) {
          console.error("No token found for fetching history.");
          return;
        }
        const response = await fetch("http://localhost:8000/api/history", {
          method: "GET",
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json",
            "Authorization": "Bearer " + token,
          },
          credentials: "include",
        });
        if (response.ok) {
          const history = await response.json();
          const formattedMessages = history.flatMap((item) => [
            {
              sender: "user",
              text: item.user_message,
              created_at: new Date(item.created_at).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }) + ' ' + new Date(item.created_at).toLocaleDateString(),
            },
            {
              sender: "bot",
              text: item.ai_response,
              created_at: new Date(item.created_at).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }) + ' ' + new Date(item.created_at).toLocaleDateString(),
            },
          ]);
          this.messages = formattedMessages;
        } else {
          console.error("Failed to fetch chat history:", response.status, await response.text());
           if (response.status === 401) {
            localStorage.removeItem("token");
            this.$router.push("/login");
          }
        }
      } catch (error) {
        console.error("Error fetching chat history:", error);
      }
    },
    async logoutUser() {
      try {
        const token = localStorage.getItem('token'); 

        if (!token) {
          console.error('No token found, cannot logout.');
          // Mungkin sudah logout atau token tidak pernah ada
          this.$router.push('/login'); // Redirect ke login untuk jaga-jaga
          return;
        }

        const response = await fetch('http://localhost:8000/api/logout', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            'Authorization': `Bearer ${token}`
          },
          credentials: 'include', // Jika server Anda memerlukan credentials
          // Body bisa kosong jika backend tidak memerlukannya untuk logout
          // body: JSON.stringify({}), 
        });

        if (response.ok) {
          const data = await response.json();
          console.log(data.message || "Successfully logged out"); 
        } else {
          // Bahkan jika logout gagal di server (misal token sudah expired), 
          // kita tetap ingin membersihkan sisi client dan redirect.
          console.warn('Logout request failed or token was invalid. Clearing client-side token.');
        }

      } catch (error) {
        console.error('Logout failed:', error);
      } finally {
        // Selalu hapus token dari client dan redirect, apapun hasil dari server
        localStorage.removeItem('token');
        this.$router.push('/login'); 
      }
    }
  },
  mounted() {
    this.fetchChatHistory();
  },
};
</script>

<style scoped>
html, body {
  height: 100%;
  margin: 0;
}

.chat-wrapper {
  display: flex;
  flex-direction: column;
  height: 80vh;
  max-width: 800px;
  margin: auto;
  margin-top: 50px;
  border: 1px solid #ddd;
  font-family: sans-serif;
  background: #fff;
}

.chat-window {
  flex-grow: 1;
  padding: 1rem;
  background: #f5f5f5;
  overflow-y: auto;
}

.message {
  display: flex;
  margin-bottom: 10px;
}

.message.user {
  justify-content: flex-end;
}

.message.bot {
  justify-content: flex-start;
}

.bubble {
  max-width: 70%;
  padding: 10px 15px;
  border-radius: 20px;
  word-wrap: break-word;
  text-align: left;
  position: relative;
}

.timestamp {
  font-size: 0.7rem;
  color: #888;
  margin-top: 5px;
  text-align: right;
}

.message.user .bubble .timestamp {
  text-align: right;
}

.message.bot .bubble .timestamp {
  text-align: left;
}

.message.user .bubble {
  background: #dcf8c6;
  align-self: flex-end;
}

.message.bot .bubble {
  background: #ffffff;
  border: 1px solid #ddd;
}

.input-area {
  display: flex;
  padding: 1rem;
  border-top: 1px solid #ddd;
  background: #fff;
}

.input-area input {
  flex-grow: 1;
  padding: 0.5rem 1rem;
  border: 1px solid #ccc;
  border-radius: 20px;
  margin-right: 10px;
}

.input-area button {
  padding: 0.5rem 1rem;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
}

.input-area button:disabled {
  background: #a5d6c8;
  cursor: not-allowed;
}

.header-bar {
  display: flex;
  justify-content: flex-end; /* Posisikan tombol logout ke kanan */
  padding: 0.5rem 1rem;
  background-color: #f5f5f5; /* Sesuaikan dengan tema Anda */
  border-bottom: 1px solid #ddd;
}

.logout-button {
  padding: 0.5rem 1rem;
  background-color: #e74c3c; /* Warna merah untuk logout */
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9rem;
}

.logout-button:hover {
  background-color: #c0392b;
}
</style> 
