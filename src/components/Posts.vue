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
      currentUser: {},
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
    // INI BAGIAN COMMENT ========================================================================================================================
    isCommentAuthor(comment) {
      console.log('Pengguna ID:', this.currentUser.id);
      console.log('Commentator ID:', comment.commentator ? comment.commentator.id : null);

      return (
        this.token &&
        comment.commentator &&
        comment.commentator.id === this.currentUser.id
      );
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
            const newComment = response.data.data;
            post.comments.push(newComment);
            post.newComment = "";
          })
          .catch((error) => {
            console.error('Error adding comment:', error);
          });
      } else {
        console.error('Comment content is required.');
      }
    },
    editComment(commentId) {
      const comment = this.findCommentById(commentId);
      this.editedCommentId = commentId;
      this.editCommentContent = comment.comments_content;
    },
    saveEditedComment(commentId) {
      axios.patch(`http://127.0.0.1:8000/api/comment/${commentId}`, {
        comments_content: this.editCommentContent,
      }, {
        headers: {
          Authorization: `Bearer ${this.token}`,
        },
      })
        .then(response => {
          const editedComment = response.data.data;
          const postIndex = this.posts.findIndex(post => post.comments.some(comment => comment.id === commentId));
          const commentIndex = this.posts[postIndex].comments.findIndex(comment => comment.id === commentId);
          this.posts[postIndex].comments.splice(commentIndex, 1, editedComment);
          this.editedCommentId = null;
          this.editCommentContent = '';
        })
        .catch(error => {
          console.error('Error editing comment:', error);
        });
    },

    cancelEdit() {
      this.editedCommentId = null;
      this.editCommentContent = '';
    },

    deleteComment(commentId) {
      axios.delete(`http://127.0.0.1:8000/api/comment/${commentId}`)
        .then(response => {
          const post = this.posts.find(post => post.comments.some(comment => comment.id === commentId));
          const commentIndex = post.comments.findIndex(comment => comment.id === commentId);
          post.comments.splice(commentIndex, 1);

        })
        .catch(error => {
          console.error('Gagal Menghapus Comment: ', error);
        });
    },

    // INI BAGIAN COMMENT ========================================================================================================================

  },
  mounted() {
    this.fetchData();
  }
};
</script>
  
<!-- saveEditedComment() {
  if (this.editedCommentId !== null) {
    const editedComment = this.findCommentById(this.editedCommentId);

    if (editedComment) {
      axios
        .patch(`http://127.0.0.1:8000/api/comment/${this.editedCommentId}`,
          {
            comments_content: this.editCommentContent,
          },
          {
            headers: {
              Authorization: `Bearer ${this.token}`,
            },
          }
        )
        .then((response) => {
          editedComment.comments_content = response.data.data.comments_content;

          this.editedCommentId = null;
          this.editCommentContent = '';

        })
        .catch((error) => {
          console.error('Error saat update comment', error);
        })
    } else {
      console.error('Comment tidak ditemukan', error);
    }
  }else{
    console.error('Tidak ada Comment yang dipilih', error);
}

},
findCommentById() {
  for (const post of this.posts) {
    const foundComment = post.comment.find((comment) => comment.id === commentId);
    if (foundComment) {
      return foundComment;
    }
  }
  return null;
}, -->