<template>
    <div>
        <h1>Login Page</h1>
        <form @submit.prevent="login">
            <input type="text" v-model="email" placeholder="Email" />
            <input type="password" v-model="password" placeholder="Password" />
            <button type="submit">Login</button>
        </form>
    </div>
</template>

<script>
import axios from "axios";

export default {
    data() {
        return {
            email: "",
            password: "",
        };
    },
    methods: {
        login() {
            axios.post("http://127.0.0.1:8000/api/login", {
                email: this.email,
                password: this.password,
            })
                .then((response) => {
                    console.log("Login success", response.data);

                    // Simpan token Bearer di localStorage
                    localStorage.setItem("token", response.data.token); // Pastikan token disimpan dengan benar
                    console.log("Token disimpan di localStorage:", response.data.token); // Tampilkan token yang disimpan

                    // Coba ambil token dari localStorage dan tampilkan
                    const tokenFromLocalStorage = localStorage.getItem("token");
                    console.log("Token dari localStorage:", tokenFromLocalStorage);

                    // Cek jika token berhasil disimpan
                    if (tokenFromLocalStorage) {
                        this.$router.push("/posts");
                    } else {
                        console.error("Gagal menyimpan token di localStorage");
                    }
                })
                .catch((error) => {
                    console.error("Login failure", error);
                });
        },
    },
};
</script>
