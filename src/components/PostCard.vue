<template>
  <div class="post-card" @click="goToPost">
    <div class="post-header">
      <h3>{{ post.title }}</h3>
      <span class="username">{{ post.user.username }}</span> 
      <span class="date">{{ formattedDate }}</span>
    </div>
    <p class="post-content">{{ post.content }}</p>
  </div>
</template>

<script>
import { format } from "date-fns";

export default {
  name: "PostCard",
  props: {
    post: Object
  },
  computed: {
    formattedDate() {
      return format(new Date(this.post.createdAt), "PPP p");
    }
  },
  methods: {
    goToPost() {
      this.$router.push(`/posts/${this.post._id}`);
    }
  }
};
</script>


<style scoped>
.post-card {
  background-color: white;
  padding: 1rem 1.2rem;
  border-radius: 15px;
  box-shadow: 0 6px 15px rgba(0,0,0,0.15);
  width: 100%;
  cursor: pointer;
  transition: transform 0.2s ease;
  max-width: 500px; 
  color: #333;
  text-align: left;
}

.post-header h3,
.username,
.date,
.post-content {
  text-align: left;
}

.post-header { display: flex; flex-direction: column; margin-bottom: 0.5rem; }
.username { font-weight: bold; color: #2a2a72; }
.date { font-size: 0.8rem; color: #666; }
.post-content { margin: 0.5rem 0; word-wrap: break-word; }
.post-actions { display: flex; gap: 0.5rem; margin-top: 0.5rem; }
.edit-btn, .delete-btn { padding: 5px 10px; border-radius: 5px; border: none; cursor: pointer; }
.edit-btn { background-color: #2a2a72; color: white; }
.delete-btn { background-color: #d33; color: white; }

@media (max-width: 768px) { .post-card { padding: 0.8rem 1rem; } }
</style>