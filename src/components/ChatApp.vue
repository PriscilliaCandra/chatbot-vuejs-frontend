<!-- hilangin scroll di luar -->

<!-- munculin tanggalnya dan history -->

<template>
  <div class="chat-wrapper">
    <div class="chat-window" ref="chatWindow">
      <div
        v-for="(msg, index) in messages"
        :key="index"
        :class="['message', msg.sender]"
      >
        <div class="bubble">{{ msg.text }}</div>
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
        const res = await fetch("http://localhost:8000/api/chat", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ text }),
        });

        console.log(res); // Debug response status

        if (!res.ok) {
          throw new Error("Failed to fetch: " + res.status);
        }

        const data = await res.json();
        console.log(data); // Debug response data

        this.messages.push({
          sender: "bot",
          text: data.result || "Gagal mendapatkan respons",
        });
      } catch (error) {
        console.error("Error during request:", error); // Debug error
        this.messages.push({ sender: "bot", text: "Terjadi kesalahan" });
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
  },
};
</script>

<style scoped>
html, body {
  overflow: hidden;
  height: 100%;
  margin: 0;
}

.chat-wrapper {
  display: flex;
  flex-direction: column;
  height: 80vh;
  max-width: 1000px;
  margin: auto;
  border: 1px solid #ddd;
  font-family: sans-serif;
  background: #fff;
}

.chat-window {
  flex-grow: 1;
  padding: 1rem;
  background: #f5f5f5;
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
</style> 
