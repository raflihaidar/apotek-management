<script setup>
import { ref, computed } from "vue"

const tab = ref("jual")

const data = ref(JSON.parse(localStorage.getItem("apotek_data") || "[]"))

const jenisKeluar = [
  "Gaji Karyawan",
  "Beli Perlengkapan",
  "Listrik/Air",
  "Sewa",
  "Lain-lain"
]

const today = () =>
  new Date().toISOString().slice(0, 10)

const form = ref({
  jual: { tanggal: today(), nilai: "", ket: "" },
  beli: { tanggal: today(), nilai: "", ket: "" },
  keluar: {
    tanggal: today(),
    nilai: "",
    ket: "",
    jenis: "Gaji Karyawan"
  }
})

const save = () =>
  localStorage.setItem("apotek_data", JSON.stringify(data.value))

const simpan = (tipe) => {
  console.log("test")
  let f =
    tipe === "jual"
      ? form.value.jual
      : tipe === "pembelian"
        ? form.value.beli
        : form.value.keluar

  if (!f.nilai || Number(f.nilai) <= 0) return

  data.value.unshift({
    id: Date.now(),
    tipe,
    tanggal: f.tanggal,
    nilai: Number(f.nilai),
    ket: f.ket,
    jenis: tipe === "pengeluaran"
      ? form.value.keluar.jenis
      : ""
  })

  save()

  if (tipe === "jual")
    form.value.jual = {
      tanggal: today(),
      nilai: "",
      ket: ""
    }

  if (tipe === "pembelian")
    form.value.beli = {
      tanggal: today(),
      nilai: "",
      ket: ""
    }

  if (tipe === "pengeluaran")
    form.value.keluar.nilai = ""
}

const hapus = (id) => {
  data.value = data.value.filter(
    (d) => d.id !== id
  )
  save()
}

const filterData = (tipe) =>
  data.value.filter((d) => d.tipe === tipe)

const totalJual = computed(() =>
  filterData("jual").reduce(
    (a, b) => a + b.nilai,
    0
  )
)

const totalBeli = computed(() =>
  filterData("pembelian").reduce(
    (a, b) => a + b.nilai,
    0
  )
)

const totalKeluar = computed(() =>
  filterData("pengeluaran").reduce(
    (a, b) => a + b.nilai,
    0
  )
)

const labaNet = computed(
  () =>
    totalJual.value -
    totalBeli.value -
    totalKeluar.value
)

const formatRp = (n) =>
  new Intl.NumberFormat("id-ID").format(n)
</script>

<template>
  <div class="max-w-md mx-auto min-h-screen bg-slate-900 text-white p-5 space-y-4">

    <!-- HEADER -->
    <div class="space-y-1">
      <p class="text-xs uppercase text-slate-400">
        Apotek Manager
      </p>

      <h1 class="text-2xl font-extrabold bg-gradient-to-r from-white to-indigo-400 bg-clip-text text-transparent">
        Kas & Transaksi
      </h1>
    </div>


    <!-- SUMMARY -->
    <div class="flex gap-2 overflow-x-auto">

      <div class="bg-slate-800 rounded-xl px-4 py-2 min-w-[120px]">
        <p class="text-xs text-slate-400">
          Penjualan
        </p>

        <p class="text-green-400 font-bold">
          Rp {{ formatRp(totalJual) }}
        </p>
      </div>


      <div class="bg-slate-800 rounded-xl px-4 py-2 min-w-[120px]">
        <p class="text-xs text-slate-400">
          Pembelian
        </p>

        <p class="text-blue-400 font-bold">
          Rp {{ formatRp(totalBeli) }}
        </p>
      </div>


      <div class="bg-slate-800 rounded-xl px-4 py-2 min-w-[120px]">
        <p class="text-xs text-slate-400">
          Pengeluaran
        </p>

        <p class="text-red-400 font-bold">
          Rp {{ formatRp(totalKeluar) }}
        </p>
      </div>

    </div>


    <!-- TAB -->
    <div class="grid grid-cols-4 gap-2">

      <button @click="tab = 'jual'" :class="tab === 'jual'
        ? 'bg-green-500'
        : 'bg-slate-800'" class="rounded-lg py-2 text-xs font-bold">
        Jual
      </button>

      <button @click="tab = 'beli'" :class="tab === 'beli'
        ? 'bg-blue-500'
        : 'bg-slate-800'" class="rounded-lg py-2 text-xs font-bold">
        Beli
      </button>

      <button @click="tab = 'keluar'" :class="tab === 'keluar'
        ? 'bg-red-500'
        : 'bg-slate-800'" class="rounded-lg py-2 text-xs font-bold">
        Keluar
      </button>

      <button @click="tab = 'rekap'" :class="tab === 'rekap'
        ? 'bg-purple-500'
        : 'bg-slate-800'" class="rounded-lg py-2 text-xs font-bold">
        Rekap
      </button>

    </div>


    <!-- FORM PENJUALAN -->
    <div v-if="tab === 'jual'" class="bg-slate-800 rounded-xl p-4 space-y-3">

      <input type="date" v-model="form.jual.tanggal" class="input" />

      <input type="number" placeholder="Nilai" v-model="form.jual.nilai" class="input" />

      <textarea placeholder="Keterangan" v-model="form.jual.ket" class="input" />

      <button @click="simpan('jual')" class="btn-green">
        Simpan Penjualan
      </button>

    </div>


    <!-- REKAP -->
    <div v-if="tab === 'rekap'" class="bg-slate-800 rounded-xl p-4 space-y-3">

      <div class="flex justify-between">
        <span>Penjualan</span>
        <span class="text-green-400">
          Rp {{ formatRp(totalJual) }}
        </span>
      </div>

      <div class="flex justify-between">
        <span>Pembelian</span>
        <span class="text-blue-400">
          Rp {{ formatRp(totalBeli) }}
        </span>
      </div>

      <div class="flex justify-between">
        <span>Pengeluaran</span>
        <span class="text-red-400">
          Rp {{ formatRp(totalKeluar) }}
        </span>
      </div>

      <hr class="border-slate-700" />

      <div class="flex justify-between font-bold">
        <span>Laba Bersih</span>

        <span :class="labaNet >= 0
            ? 'text-green-400'
            : 'text-red-400'
          ">
          Rp {{ formatRp(labaNet) }}
        </span>

      </div>

    </div>

  </div>
</template>
