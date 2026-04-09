<script setup>
import { ref, computed } from "vue"
import * as XLSX from "xlsx"

const tab = ref("jual")
const editId = ref(null)

const data = ref(JSON.parse(localStorage.getItem("apotek_data") || "[]"))

const jenisKeluar = [
  "Gaji Karyawan",
  "Beli Perlengkapan",
  "Listrik/Air",
  "Sewa",
  "Lain-lain"
]

const today = () => new Date().toISOString().slice(0, 10)

const form = ref({
  jual: { tanggal: today(), nilai: "", ket: "" },
  pembelian: { tanggal: today(), nilai: "", ket: "" },
  pengeluaran: {
    tanggal: today(),
    nilai: "",
    ket: "",
    jenis: "Gaji Karyawan"
  }
})

const save = () =>
  localStorage.setItem("apotek_data", JSON.stringify(data.value))

// SIMPAN DATA
const simpan = (tipe) => {
  let f = form.value[tipe]

  if (!f.nilai || Number(f.nilai) <= 0) return

  data.value.unshift({
    id: Date.now(),
    tipe,
    tanggal: f.tanggal,
    nilai: Number(f.nilai),
    ket: f.ket,
    jenis: tipe === "pengeluaran" ? f.jenis : ""
  })

  save()

  form.value[tipe] = {
    tanggal: today(),
    nilai: "",
    ket: "",
    ...(tipe === "pengeluaran" && { jenis: "Gaji Karyawan" })
  }
}

// EXPORT EXCEL
const exportExcel = () => {
  if (data.value.length === 0) {
    alert("Data kosong!")
    return
  }

  const rows = data.value.map((d) => ({
    tanggal: d.tanggal,
    tipe: d.tipe,
    nilai: d.nilai,
    keterangan: d.ket,
    jenis: d.jenis
  }))

  const ws = XLSX.utils.json_to_sheet(rows)
  const wb = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(wb, ws, "Laporan")

  XLSX.writeFile(wb, "laporan_apotek.xlsx")
}

// HAPUS
const hapus = (id) => {
  if (!confirm("Yakin hapus data?")) return
  data.value = data.value.filter((d) => d.id !== id)
  save()
}

// EDIT
const mulaiEdit = (item) => {
  editId.value = item.id
}

const simpanEdit = () => {
  editId.value = null
  save()
}

// TOTAL
const totalJual = computed(() =>
  data.value.filter(d => d.tipe === "jual").reduce((a, b) => a + b.nilai, 0)
)

const totalBeli = computed(() =>
  data.value.filter(d => d.tipe === "pembelian").reduce((a, b) => a + b.nilai, 0)
)

const totalKeluar = computed(() =>
  data.value.filter(d => d.tipe === "pengeluaran").reduce((a, b) => a + b.nilai, 0)
)

const labaNet = computed(
  () => totalJual.value - totalBeli.value - totalKeluar.value
)

const formatRp = (n) =>
  new Intl.NumberFormat("id-ID").format(n)
</script>

<template>
  <div class="min-h-screen bg-slate-900 text-white">

    <div class="max-w-6xl mx-auto p-6 space-y-6">

      <!-- HEADER -->
      <div>
        <h1 class="text-2xl font-bold">Apotek Manager</h1>
        <p class="text-slate-400 text-sm">Pembukuan + Export Excel</p>
      </div>

      <!-- SUMMARY -->
      <div class="grid grid-cols-2 md:grid-cols-4 gap-4">

        <div class="bg-slate-800 p-4 rounded-xl">
          <p class="text-xs text-slate-400">Penjualan</p>
          <p class="text-green-400 font-bold">Rp {{ formatRp(totalJual) }}</p>
        </div>

        <div class="bg-slate-800 p-4 rounded-xl">
          <p class="text-xs text-slate-400">Pembelian</p>
          <p class="text-blue-400 font-bold">Rp {{ formatRp(totalBeli) }}</p>
        </div>

        <div class="bg-slate-800 p-4 rounded-xl">
          <p class="text-xs text-slate-400">Pengeluaran</p>
          <p class="text-red-400 font-bold">Rp {{ formatRp(totalKeluar) }}</p>
        </div>

        <div class="bg-slate-800 p-4 rounded-xl">
          <p class="text-xs text-slate-400">Laba</p>
          <p :class="labaNet >= 0 ? 'text-green-400' : 'text-red-400'" class="font-bold">
            Rp {{ formatRp(labaNet) }}
          </p>
        </div>

      </div>

      <!-- TAB -->
      <div class="flex gap-2">
        <button @click="tab = 'jual'" class="btn" :class="tab === 'jual' && 'active'">Jual</button>
        <button @click="tab = 'pembelian'" class="btn" :class="tab === 'pembelian' && 'active'">Beli</button>
        <button @click="tab = 'pengeluaran'" class="btn" :class="tab === 'pengeluaran' && 'active'">Keluar</button>
        <button @click="tab = 'rekap'" class="btn" :class="tab === 'rekap' && 'active'">Rekap</button>
      </div>

      <!-- FORM -->
      <div class="bg-slate-800 p-5 rounded-xl space-y-3">

        <div v-if="tab === 'jual'">
          <input type="date" v-model="form.jual.tanggal" class="input" />
          <input type="number" v-model="form.jual.nilai" placeholder="Nilai" class="input" />
          <input type="text" v-model="form.jual.ket" placeholder="Keterangan" class="input" />
          <button @click="simpan('jual')" class="btn-green">Simpan</button>
        </div>

        <div v-if="tab === 'pembelian'">
          <input type="date" v-model="form.pembelian.tanggal" class="input" />
          <input type="number" v-model="form.pembelian.nilai" class="input" />
          <input type="text" v-model="form.pembelian.ket" class="input" />
          <button @click="simpan('pembelian')" class="btn-blue">Simpan</button>
        </div>

        <div v-if="tab === 'pengeluaran'">
          <input type="date" v-model="form.pengeluaran.tanggal" class="input" />
          <select v-model="form.pengeluaran.jenis" class="input">
            <option v-for="j in jenisKeluar" :key="j">{{ j }}</option>
          </select>
          <input type="number" v-model="form.pengeluaran.nilai" class="input" />
          <input type="text" v-model="form.pengeluaran.ket" class="input" />
          <button @click="simpan('pengeluaran')" class="btn-red">Simpan</button>
        </div>

        <!-- REKAP -->
        <div v-if="tab === 'rekap'" class="space-y-4">

          <div class="bg-slate-700 p-4 rounded-lg flex justify-between">
            <span>Export Laporan</span>
            <button @click="exportExcel" class="bg-green-500 px-3 py-1 rounded">
              Export
            </button>
          </div>

          <p>Penjualan: Rp {{ formatRp(totalJual) }}</p>
          <p>Pembelian: Rp {{ formatRp(totalBeli) }}</p>
          <p>Pengeluaran: Rp {{ formatRp(totalKeluar) }}</p>
          <p class="font-bold">Laba: Rp {{ formatRp(labaNet) }}</p>

        </div>

      </div>

      <!-- HISTORY -->
      <div class="bg-slate-800 p-5 rounded-xl space-y-3">

        <h2 class="font-bold">History Transaksi</h2>

        <div v-for="item in data" :key="item.id" class="bg-slate-700 p-3 rounded">

          <!-- VIEW -->
          <div v-if="editId !== item.id" class="flex justify-between">
            <div>
              <p :class="{
                'text-green-400': item.tipe === 'jual',
                'text-blue-400': item.tipe === 'pembelian',
                'text-red-400': item.tipe === 'pengeluaran'
              }">
                {{ item.tipe }} - Rp {{ formatRp(item.nilai) }}
              </p>
              <p class="text-xs">{{ item.tanggal }} | {{ item.ket }}</p>
            </div>

            <div class="flex gap-2">
              <button @click="mulaiEdit(item)" class="bg-yellow-500 px-2 rounded text-xs">Edit</button>
              <button @click="hapus(item.id)" class="bg-red-500 px-2 rounded text-xs">Hapus</button>
            </div>
          </div>

          <!-- EDIT -->
          <div v-else class="space-y-2">
            <input type="date" v-model="item.tanggal" class="input" />
            <input type="number" v-model="item.nilai" class="input" />
            <input type="text" v-model="item.ket" class="input" />
            <button @click="simpanEdit()" class="bg-green-500 px-3 py-1 rounded">
              Simpan
            </button>
          </div>

        </div>

      </div>

    </div>
  </div>
</template>

<style>
.input {
  width: 100%;
  padding: 8px;
  border-radius: 6px;
  background: #1e293b;
  margin-bottom: 8px;
}

.btn {
  padding: 8px 12px;
  background: #1e293b;
  border-radius: 6px;
}

.btn.active {
  background: #6366f1;
}

.btn-green {
  background: #22c55e;
  padding: 8px;
  border-radius: 6px;
}

.btn-blue {
  background: #3b82f6;
  padding: 8px;
  border-radius: 6px;
}

.btn-red {
  background: #ef4444;
  padding: 8px;
  border-radius: 6px;
}
</style>