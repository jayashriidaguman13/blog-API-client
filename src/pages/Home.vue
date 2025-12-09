<template>
  <div class="home">
    <section class="landing">
      <div class="content">
        <h1>Astral Archive</h1>
        <p>Welcome to the Astral Archive, a place where the mysteries of the universe are explored through the written word.</p>
        <div class="landing-buttons">
          <button v-if="!isLoggedIn" class="join-btn" @click="$router.push('/register')">Join Now</button>
          <button v-if="!isLoggedIn" class="login-btn" @click="$router.push('/login')">Login</button>
          <button v-if="isLoggedIn" class="login-btn" @click="handleLogout">Logout</button>
        </div>
      </div>
    </section>

    <section class="posts-area">
      <div class="content">
        <div class="create-post-section">
          <button v-if="!isAdmin" class="join-btn" @click="isLoggedIn ? (showCreate = !showCreate) : $router.push('/login')">
            {{ showCreate ? 'Cancel' : 'Create Post' }}
          </button>
          <div v-if="showCreate" class="create-post-card">
            <input v-model="newPost.title" placeholder="Title" />
            <textarea v-model="newPost.content" placeholder="Content"></textarea>
            <div class="create-post-buttons">
              <button class="join-btn" @click="handleCreatePost">Post</button>
              <button class="join-btn cancel-btn" @click="showCreate = false">Cancel</button>
            </div>
          </div>
        </div>

        <div class="posts-container">
          <PostCard
            v-for="post in posts"
            :key="post._id"
            :post="post"
            :currentUserId="currentUserId"
            :isAdmin="isAdmin"
          />
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import api from "../api.js";
import PostCard from "../components/PostCard.vue";
import notyf from "../notyf.js";
import { jwtDecode } from "jwt-decode";

export default {
  name: "Home",
  components: { PostCard },
  data() {
    return {
      posts: [],
      showCreate: false,
      newPost: { title: "", content: "" },
      currentUserId: null,
      isAdmin: false,
      isLoggedIn: !!localStorage.getItem("token"),
    };
  },
  methods: {
    async fetchPosts() {
      try {
        const res = await api.get("/posts");
        this.posts = res.data.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
      } catch (err) {
        notyf.error("Failed to fetch posts");
      }
    },
    handleLogout() {
      localStorage.removeItem("token");
      this.isLoggedIn = false;
      this.currentUserId = null;
      this.isAdmin = false;
      notyf.success("Logged out successfully");
      setTimeout(() => this.$router.push("/"), 500);
    },
    async handleCreatePost() {
      if (!this.newPost.title || !this.newPost.content) {
        notyf.error("Title and content are required");
        return;
      }
      try {
        const token = localStorage.getItem("token");
        const res = await api.post("/posts", this.newPost, {
          headers: { Authorization: `Bearer ${token}` },
        });
        this.posts.unshift(res.data);
        this.newPost.title = "";
        this.newPost.content = "";
        this.showCreate = false;
        notyf.success("Post created successfully");
      } catch (err) {
        notyf.error(err.response?.data?.message || "Failed to create post");
      }
    },
  },
  mounted() {
    this.fetchPosts();
    if (this.isLoggedIn) {
      try {
        const token = localStorage.getItem("token");
        const decoded = jwtDecode(token);
        this.currentUserId = decoded.id;
        this.isAdmin = decoded.isAdmin;
      } catch (err) {
        console.error("Token decode failed", err);
      }
    }
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;500&family=Quicksand:wght@300;500&display=swap');

.home {
  background: url('/images/galaxy.jpg') no-repeat center center fixed;
  background-size: cover;
  color: white;
  text-align: center;
  font-family: 'Quicksand', sans-serif;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow: hidden;
}

.landing {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  padding: 0 1rem;
}

.posts-area {
  min-height: 100vh;
  padding: 2rem 1rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.posts-area .join-btn {
  margin-bottom: 1rem;
}

h1 {
  font-family: 'Orbitron', sans-serif;
  font-size: 5rem;
  margin-bottom: 10px;
  color: whitesmoke;
}

p {
  font-size: 1.25rem;
  margin-bottom: 20px;
  color: whitesmoke;
  max-width: 800px;
}

.back-btn,
.login-btn,
.join-btn {
  background-color: #2a2a72;
  color: white;
  padding: 10px 20px;
  font-size: 1rem;
  cursor: pointer;
  border: none;
  border-radius: 5px;
  z-index: 2;
}

.login-btn {
  position: absolute;
  top: 20px;
  right: 20px;
}

.posts-container {
  display: flex;
  flex-direction: column; 
  gap: 1.5rem;
  align-items: center; 
  width: 100%;
  max-width: 600px; 
}

.create-post-card {
  display: flex;
  flex-direction: column;
  margin-top: 1rem;
  background-color: rgba(255, 255, 255, 0.1);
  padding: 1rem;
  border-radius: 10px;
  width: 100%;
  max-width: 600px;
}

.create-post-card input,
.create-post-card textarea {
  width: 100%;
  margin-bottom: 0.75rem;
  padding: 8px;
  border-radius: 5px;
  border: none;
}

.create-post-buttons {
  display: flex;
  gap: 0.5rem;
  justify-content: flex-end;
}

.cancel-btn {
  background-color: #7f7faa;
}

@media (max-width: 768px) {
  h1 {
    font-size: 3rem;
  }
  p {
    font-size: 1rem;
  }
  .landing {
    padding: 0 1rem;
  }
  .posts-area {
    padding: 1.5rem 1rem;
  }
}

@media (max-width: 480px) {
  h1 {
    font-size: 2.2rem;
  }
  p {
    font-size: 0.9rem;
  }
  .landing {
    padding: 0 0.5rem;
  }
  .posts-area {
    padding: 1rem 0.5rem;
  }
}
</style>