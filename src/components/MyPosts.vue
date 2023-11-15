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
                <button @click="deletePost(post.id)">Hapus</button>
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
            const baseUrl = 'http://localhost:8000';
            return `${baseUrl}/storage/posts/${imageFileName}`;
        },
        fetchData() {
            if (this.token) {
                axios
                    .get("http://127.0.0.1:8000/api/myposts", {
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
        deletePost(postId) {
            if (this.token) {
                axios
                    .delete(`http://127.0.0.1:8000/api/posts/${postId}`, {
                        headers: {
                            Authorization: `Bearer ${this.token}`,
                        },
                    })
                    .then(() => {
                        // If deletion is successful, update the posts array
                        this.posts = this.posts.filter((post) => post.id !== postId);
                    })
                    .catch((error) => {
                        console.error("Error deleting post:", error);
                    });
            } else {
                console.error("Token tidak ditemukan. Silakan login terlebih dahulu.");
            }
        },
    },
    mounted() {
        this.fetchData();
    },
};
</script>
  