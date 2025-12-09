<template>
  <div class="comment-card">
    <div class="comment-header">
      <span class="username">{{ comment.user?.username || comment.user.username || "Unknown" }}</span>
      <span class="date">{{ formattedDate }}</span>
    </div>
    <p class="comment-text">{{ comment.text }}</p>
    <div v-if="canDelete(comment.user?._id || comment.userId)" class="comment-actions">
      <button class="delete-btn" @click="confirmDelete">Delete</button>
    </div>
  </div>
</template>

<script>
import notyf from "../notyf.js";
import Swal from "sweetalert2";
import { format } from "date-fns";

export default {
  name: "CommentsCard",
  props: {
    comment: Object,
    currentUserId: String,
    isAdmin: Boolean
  },
  computed: {
    formattedDate() {
      return format(new Date(this.comment.createdAt), "PPP p");
    }
  },
  methods: {
    confirmDelete() {
      Swal.fire({
        title: "Are you sure?",
        text: "This comment will be permanently deleted.",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#d33",
        cancelButtonColor: "#3085d6",
        confirmButtonText: "Yes, delete it!"
      }).then(async (result) => {
        if (result.isConfirmed) {
          this.$emit("delete", this.comment._id);
          notyf.success("Comment deleted successfully");
        }
      });
    }
  }
};
</script>

<style scoped>
.comment-card {
  background-color: white;
  padding: 1rem;
  border-radius: 10px;
  margin-bottom: 1rem;
  width: 100%;
  box-shadow: 0 4px 10px rgba(0,0,0,0.15);
  text-align: left; /* Align text to the left */
}

.comment-header {
  display: flex;
  justify-content: space-between;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.username {
  color: #2a2a72;
}

.date {
  font-size: 0.85rem;
  color: #666;
}

.comment-text {
  margin-bottom: 0.5rem;
}

.comment-actions {
  display: flex;
  gap: 0.5rem;
}

.delete-btn {
  padding: 5px 10px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
}

.delete-btn {
  background-color: #d33;
  color: white;
}

@media (max-width: 768px) {
  .comment-card {
    padding: 0.8rem;
  }
  .comment-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.2rem;
  }
  .comment-actions {
    flex-direction: column;
    gap: 0.3rem;
  }
}
</style>
