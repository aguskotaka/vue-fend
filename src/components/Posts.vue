<template>
  <div>
    <h1>Daftar Post</h1>
    <p>Token: {{ token }}</p>
    <ul>
      <li v-for="post in posts" :key="post.id">
        <img :src="getImageUrl(post.image)" alt="Post Image" />
        <h3>{{ post.title }}</h3>
        <p>{{ post.news }}</p>
        <p>Author: {{ post.writer.username }}</p>
        <p>Total Comments: {{ post.comment_total }}</p>
        <form @submit.prevent="addComment(post.id)">
          <input v-model="post.newComment" type="text" placeholder="Add a comment..." />
          <button type="submit">Submit</button>
        </form>
        <ul>
          <li v-for="comment in post.comments" :key="comment.id">
            <p>{{ comment.comments_content }}</p>
            <p v-if="comment.commentator">Commentator: {{ comment.commentator.username }}</p>
            <p v-else>Commentator: </p>
            
            <!-- Display edit and delete options only if the current user is the comment author -->
            <div v-if="isCommentAuthor(comment)">
              <button @click="editComment(comment.id)">Edit</button>
              <button @click="deleteComment(comment.id)">Delete</button>
            </div>
          </li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      email: "",
      password: "",
      token: localStorage.getItem("token"),
      posts: [],
    };
  },
  methods: {
    getImageUrl(imageFileName) {
      // Replace this with the base URL of your Laravel server
      const baseUrl = 'http://localhost:8000';

      // Construct the full URL for the image
      return `${baseUrl}/storage/posts/${imageFileName}`;
    },
    fetchData() {
      if (this.token) {
        axios
          .get("http://127.0.0.1:8000/api/posts", {
            headers: {
              Authorization: `Bearer ${this.token}`,
            },
          })
          .then((response) => {
            this.posts = response.data.data;
          })
          .catch((error) => {
            console.error("Error fetching data:", error);
          });
      } else {
        console.error("Token tidak ditemukan. Silakan login terlebih dahulu.");
      }
    },
    addComment(postId) {
      const post = this.posts.find((p) => p.id === postId);

      if (post.newComment) {
        axios
          .post(
            "http://127.0.0.1:8000/api/comment",
            {
              post_id: postId,
              comments_content: post.newComment,
            },
            {
              headers: {
                Authorization: `Bearer ${this.token}`,
              },
            }
          )
          .then((response) => {
            // Assuming the API returns the newly created comment
            const newComment = response.data.data;
            post.comments.push(newComment);
            post.newComment = ""; // Clear the comment input field
          })
          .catch((error) => {
            console.error('Error adding comment:', error);
          });
      } else {
        console.error('Comment content is required.');
      }
    },
    saveEditedComment(){
      
    }
  },
  mounted() {
  this.fetchData();
  }
};
</script>
  