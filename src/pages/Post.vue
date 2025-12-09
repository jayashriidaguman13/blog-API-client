<template>
  <div class="post">
    <div class="overlay"></div>
    <button class="back-btn" @click="$router.push('/')">Back</button>
    <div class="content">
      <div class="post-card">
        <h2>{{ post.title }}</h2>
        <p class="author">by {{ post.user?.username || "Unknown" }} on {{ formattedDate(post.createdAt) }}</p>
        <p class="post-content">{{ post.content }}</p>
        <div v-if="post.user?._id === currentUserId || isAdmin">
          <button v-if="post.user?._id === currentUserId" class="join-btn" @click="handleEditPost(post)">Edit</button>
          <button class="join-btn" @click="handleDeletePost(post._id)">Delete</button>
        </div>
      </div>

      <div class="comments-section">
        <h3>Comments</h3>
        <div v-for="comment in comments" :key="comment._id" class="comment-card">
          <p><strong>{{ comment.user?.username || "Unknown" }}</strong>: {{ comment.text }}</p>
          <div v-if="canEditOrDelete(comment.user?._id)">
            <button class="join-btn" @click="handleDeleteComment(comment._id)">Delete</button>
          </div>
        </div>
        <div v-if="!isAdmin" class="add-comment">
          <textarea v-model="newComment" placeholder="Add a comment..."></textarea>
          <button class="join-btn" @click="handleAddComment">Add Comment</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import api from "../api.js";
import notyf from "../notyf.js";
import { jwtDecode } from "jwt-decode";

export default {
  name: "SinglePost",
  data() {
    return {
      post: {},
      comments: [],
      newComment: "",
      isLoggedIn: !!localStorage.getItem("token"),
      currentUserId: null,
      isAdmin: false
    };
  },
  methods: {
    async fetchPost() {
      try {
        const res = await api.get(`/posts/${this.$route.params.id}`);
        this.post = res.data;
        this.comments = res.data.comments || [];
      } catch (err) {
        notyf.error("Failed to load post");
      }
    },
    canEditOrDelete(userId) {
      return this.isAdmin || userId === this.currentUserId;
    },
    async handleAddComment() {
      if (!this.isLoggedIn) {
        this.$router.push("/login");
        return;
      }
      if (!this.newComment) {
        notyf.error("Comment cannot be empty");
        return;
      }
      try {
        const token = localStorage.getItem("token");
        await api.post(
          `/posts/${this.post._id}/comments`,
          { text: this.newComment },
          { headers: { Authorization: `Bearer ${token}` } }
        );
        await this.fetchPost();
        this.newComment = "";
        notyf.success("Comment added");
      } catch (err) {
        notyf.error(err.response?.data?.message || "Failed to add comment");
      }
    },
    async handleDeletePost(postId) {
      if (!confirm("Are you sure you want to delete this post?")) return;
      try {
        const token = localStorage.getItem("token");
        await api.delete(`/posts/${postId}`, {
          headers: { Authorization: `Bearer ${token}` }
        });
        notyf.success("Post deleted");
        this.$router.push("/");
      } catch (err) {
        notyf.error(err.response?.data?.message || "Failed to delete post");
      }
    },
    async handleDeleteComment(commentId) {
      if (!confirm("Are you sure you want to delete this comment?")) return;
      try {
        const token = localStorage.getItem("token");
        await api.delete(`/posts/${this.post._id}/comments/${commentId}`, {
          headers: { Authorization: `Bearer ${token}` }
        });
        this.comments = this.comments.filter(c => c._id !== commentId);
        notyf.success("Comment deleted");
      } catch (err) {
        notyf.error(err.response?.data?.message || "Failed to delete comment");
      }
    },
    handleEditPost(post) {
      notyf.success("Edit functionality triggered (implement inline edit or separate page)");
    },
    formattedDate(dateStr) {
      return new Date(dateStr).toLocaleString();
    }
  },
  mounted() {
    this.fetchPost();
    if (this.isLoggedIn) {
      try {
        const decoded = jwtDecode(localStorage.getItem("token"));
        this.currentUserId = decoded.id;
        this.isAdmin = decoded.isAdmin;
      } catch (err) {
        console.error("Token decode failed", err);
      }
    }
  }
};
</script>

<style scoped>
.post {
  background: url('../public/images/post-bg.jpg') no-repeat center center fixed;
  background-size: cover;
  min-height: 100vh;
  color: white;
  font-family: 'Quicksand', sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}
.back-btn {
  position: absolute;
  top: 20px;
  left: 20px;
  z-index: 10;
  background-color: #2a2a72;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.content {
  z-index: 2;
  width: 90%;
  max-width: 800px;
  margin: 5rem 0;
}
.post-card, .comment-card {
  background-color: rgba(0,0,0,0.75);
  padding: 1rem 1.5rem;
  border-radius: 10px;
  margin-bottom: 1rem;
  text-align: left;
}
.post-card h2 {
  font-family:'Orbitron', sans-serif;
}
.post-content {
  margin-top: 0.5rem;
}
.comments-section {
  margin-top: 2rem;
}
.add-comment textarea {
  width: 100%;
  padding: 10px;
  border-radius: 5px;
  border: none;
  margin-bottom: 0.5rem;
}
.join-btn {
  background-color: #2a2a72;
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin: 2px;
}
@media(max-width:768px){
  h2 { font-size:1.5rem; }
  .post-content { font-size:1rem; }
}
@media(max-width:480px){
  h2 { font-size:1.2rem; }
  .post-content { font-size:0.9rem; }
}
</style>