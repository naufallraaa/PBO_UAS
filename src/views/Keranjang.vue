<template>
  <div class="keranjang">
    <Navbar />
    <div class="container">
      <!-- Breadcrumb -->
      <div class="row">
        <div class="col">
          <nav aria-label="breadcrumb">
            <ol class="breadcrumb">
              <li class="breadcrumb-item">
                <router-link to="/" class="text-dark">Home</router-link>
              </li>
              <li class="breadcrumb-item active" aria-current="page">Keranjang</li>
            </ol>
          </nav>
        </div>
      </div>
      
      <!-- Tabel Keranjang -->
      <div class="row">
        <div class="col">
          <h2>Keranjang <strong>Saya</strong></h2>
          <div class="table-responsive mt-3">
            <table class="table">
              <thead>
                <tr>
                  <th scope="col">No</th>
                  <th scope="col">Foto</th>
                  <th scope="col">Item</th>
                  <th scope="col">Keterangan</th>
                  <th scope="col">Jumlah Item</th>
                  <th scope="col">Harga</th>
                  <th scope="col">Total Harga</th>
                  <th scope="col"></th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(item, index) in keranjang" :key="item.id">
                  <th>{{ index + 1 }}</th>
                  <td>
                    <img :src="'../assets/images/' + item.product.gambar" class="img-fluid shadow" width="100" />
                  </td>
                  <td><strong>{{ item.product.nama }}</strong></td>
                  <td>{{ item.keterangan || '-' }}</td>
                  <td style="text-align: center">{{ item.jumlah_pemesanan }}</td>
                  <td align="right">Rp. {{ item.product.harga }}</td>
                  <td align="right"><strong>Rp. {{ parseFloat(item.product.harga) * parseInt(item.jumlah_pemesanan) }}</strong></td>
                  <td align="center">
                    <b-icon-pencil @click="editKeranjang(item)"></b-icon-pencil>
                    <b-icon-trash @click="hapusKeranjang(item.id)"></b-icon-trash>
                  </td>
                </tr>
                <tr>
                  <td colspan="6" align="right"><strong>Total Harga :</strong></td>
                  <td align="right"><strong>Rp. {{ totalHarga }}</strong></td>
                  <td></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- Form Edit -->
      <div v-if="itemDiedit" class="row">
        <div class="col">
          <h2>Edit Item</h2>
          <form @submit.prevent="simpanEdit">
            <div class="form-group">
              <label for="jumlah_pemesanan">Jumlah Pemesanan:</label>
              <input type="number" class="form-control" v-model="itemDiedit.jumlah_pemesanan" />
            </div>
            <button type="submit" class="btn btn-primary">Simpan</button>
            <button type="button" class="btn btn-secondary" @click="itemDiedit = null">Batal</button>
          </form>
        </div>
      </div>

      <!-- Form Checkout -->
      <div class="row justify-content-end">
        <div class="col-md-4">
          <form class="mt-4" @submit.prevent="checkout">
            <div class="form-group">
              <label for="nama">Nama :</label>
              <input type="text" class="form-control" v-model="pesan.nama" />
            </div>
            <div class="form-group">
              <label for="alamat">Alamat :</label>
              <input type="text" class="form-control" v-model="pesan.alamat" />
            </div>
            <button type="submit" class="btn btn-success float-right">
              <b-icon-cart></b-icon-cart> Pesan
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Navbar from "@/components/Navbar.vue";
import axios from "axios";

export default {
  name: "KeranjangCatalog",
  components: {
    Navbar,
  },
  data() {
    return {
      keranjang: [],
      pesan: {
        nama: '',
        alamat: '',
        item: []
      },
      itemDiedit: null,
    };
  },
  methods: {
    fetchData() {
      axios
        .get("http://localhost:3000/keranjang")
        .then((response) => {
          this.keranjang = response.data;
        })
        .catch((error) => {
          console.error("Error fetching data:", error);
        });
    },
    hapusKeranjang(id) {
      axios
        .delete(`http://localhost:3000/keranjang/${id}`)
        .then(() => {
          this.keranjang = this.keranjang.filter(item => item.id !== id);
          this.itemDiedit = null; // Reset itemDiedit setelah menghapus
          this.$toast.open({
            message: "Pesanan Berhasil Dihapus",
            type: "error",
            position: "top-right",
            duration: 3000,
            dismissible: true,
          });
        })
        .catch((error) => {
          this.$toast.error("Error deleting item: " + error.message);
        });
    },
    editKeranjang(item) {
      this.itemDiedit = { ...item };
    },
    simpanEdit() {
      axios.put(`http://localhost:3000/keranjang/${this.itemDiedit.id}`, this.itemDiedit)
        .then(() => {
          this.fetchData();
          this.itemDiedit = null;
          this.$toast.success("Item berhasil diupdate", {
            type: "success",
            position: "top-right",
            duration: 3000,
            dismissible: true,
          });
        })
        .catch(error => {
          console.error("Error updating item:", error);
        });
    },
    checkout() {
      if (this.pesan.nama && this.pesan.alamat) {
        this.pesan.item = this.keranjang;
        axios
          .post("http://localhost:3000/pesanan", this.pesan)
          .then(() => {
            this.keranjang.forEach(item => {
              axios.delete(`http://localhost:3000/keranjang/${item.id}`)
                .catch(error => console.error(error));
            });
            this.$router.push({ path: "/pesanan-sukses" });
            this.$toast.success("Pesanan Sukses", {
              type: "success",
              position: "top-right",
              duration: 3000,
              dismissible: true,
            });
          })
          .catch((error) => {
            console.error(error);
          });
      } else {
        this.$toast.open({
          message: "Nama dan Alamat Harus Diisi",
          type: "error",
          position: "top-right",
          duration: 3000,
          dismissible: true,
        });
      }
    }
  },
  mounted() {
    this.fetchData();
  },
  computed: {
    totalHarga() {
      return this.keranjang.reduce((total, item) => {
        return total + item.product.harga * item.jumlah_pemesanan;
      }, 0);
    },
  },
};
</script>

<style>
/* Add any custom styles here */
</style>
