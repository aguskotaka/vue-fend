<template>
    <div>
        <h1>Logout Page</h1>
        <button @click="logout">Logout</button>
    </div>
</template>

<script>
import axios from "axios";

export default {
    methods: {
        logout() {
            // Mendapatkan token Bearer dari penyimpanan lokal
            const token = localStorage.getItem("token");

            // Memeriksa apakah token ada
            if (token) {
                // Token ditemukan, lakukan permintaan logout dengan token
                axios.post("http://127.0.0.1:8000/api/logout", null, {
                    headers: {
                        Authorization: `Bearer ${token}`,
                    },
                })
                .then((response) => {
                    // Tangani respons logout berhasil
                    console.log("Logout berhasil", response.data);

                    // Hapus token dari penyimpanan lokal
                    localStorage.removeItem("token");

                    // Mengarahkan pengguna kembali ke halaman login
                    this.$router.push("/login");
                })
                .catch((error) => {
                    // Tangani kesalahan
                    console.error("Logout gagal", error);
                });
            } else {
                // Token tidak ditemukan, lakukan tindakan sesuai kebijakan Anda
                console.error("Token tidak ditemukan. Silakan login terlebih dahulu.");
            }
        },
    },
};
</script>
