# 🍫 Global Chocolate Sales Analysis 2025
### Business Intelligence & Data Warehouse — Tugas Kelompok

> **Membangun pipeline ETL dan Data Warehouse berbasis Star Schema untuk menganalisis pola penjualan cokelat global secara menyeluruh dan efisien.**

---

## 👥 Anggota Kelompok

| No | Nama | NIM |
|----|------|-----|
| 1  | [Hendri Zaidan Safitra] | [2409116013] |
| 2  | [Najmi Hafizh Mauludan Zain] | [2409116028] |
| 3  | [Muhammad Sadikin Samir] | [2409116031] |
| 4  | [Rizqy] | [2409116039] |

---

## 📦 Dataset

| Info | Detail |
|------|--------|
| **Nama Dataset** | Global Chocolate Sales 2025 |
| **Sumber** | Kaggle — [Chocolate Sales 2025 Dataset](https://www.kaggle.com/datasets/syedaeman2212/chocolate-sales-dataset-in-2025) |
| **Jumlah Record** | 500 transaksi penjualan |
| **Periode** | Januari 2025 — Desember 2025 |
| **Cakupan** | 10 negara, 7 brand, 6 jenis produk |
| **Format** | CSV |

---

## 🎯 Tujuan Proyek

Proyek ini bertujuan untuk:

- 🔄 Membangun **pipeline ETL** (Extract, Transform, Load) yang bersih dan terstruktur dari data mentah CSV
- 🏗️ Merancang **Data Warehouse** menggunakan arsitektur **Star Schema** dengan 1 Fact Table dan 5 Dimension Tables
- 📊 Menghasilkan **insight bisnis** yang actionable dari data penjualan cokelat global
- 🧠 Mengaplikasikan konsep **Business Intelligence** dalam konteks nyata menggunakan tools modern

---

## 🛠️ Teknologi yang Digunakan

| Layer | Tools |
|-------|-------|
| **Language** | Python |
| **Data Processing** | Pandas |
| **Notebook** | Google Colab |
| **Version Control** | Git & GitHub |

---

## 🔄 Alur ETL (Extract → Transform → Load)

### 1️⃣ Extract — Pengambilan Data Mentah

Data diambil langsung dari file sumber `chocolate_sales_2025_dataset.csv` yang memuat 500 baris transaksi penjualan cokelat global. Proses ekstraksi dilakukan menggunakan `pandas.read_csv()` untuk memuat data ke dalam DataFrame.

**Kolom yang tersedia di raw data:**
- `Sale_ID`, `Date`, `Brand`, `Product_Type`, `Country`
- `Sales_Channel`, `Payment_Method`, `Price_USD`, `Units_Sold`, `Revenue_USD`

---

### 2️⃣ Transform — Pembersihan & Rekayasa Data

Tahap transformasi mencakup serangkaian proses penting untuk memastikan kualitas dan konsistensi data:

**🧹 Data Cleaning:**
- Mengecek dan menangani nilai kosong (null/NaN) di seluruh kolom
- Menghapus duplikasi data berdasarkan `Sale_ID`
- Standarisasi format tanggal menggunakan `pd.to_datetime()`
- Memastikan tipe data numerik (Revenue, Units Sold) sudah benar

**🔧 Data Transformation:**
- Parsing kolom `Date` menjadi komponen `Day`, `Month`, `Year` untuk dimensi waktu
- Membuat surrogate key (`date_id`, `product_id`, `region_id`, `channel_id`, `payment_id`) untuk setiap dimensi
- Melakukan deduplication pada tiap dimensi agar tidak ada data redundan
- Mapping foreign key dari setiap dimensi ke dalam tabel fakta

**✅ Data Validation:**
- Validasi referential integrity antara fact table dan semua dimension tables
- Pengecekan range nilai numerik (Revenue & Units tidak boleh negatif)
- Verifikasi jumlah baris sebelum dan sesudah transformasi

---

### 3️⃣ Load — Penyimpanan ke Data Warehouse

Data yang sudah bersih dimuat menjadi CSV sebagai Data Warehouse. Setiap tabel dimensi di-load terlebih dahulu sebelum tabel fakta untuk menjaga integritas referensial.

---

## 🌟 Star Schema — Arsitektur Data Warehouse

Proyek ini menggunakan **Star Schema** sebagai model data warehouse, terdiri dari **1 Fact Table** di tengah yang dikelilingi oleh **5 Dimension Tables**.

```
                        ┌─────────────────┐
                        │   dim_date      │
                        │─────────────────│
                        │ PK: date_id     │
                        │ Date            │
                        │ Day             │
                        │ Month           │
                        │ Year            │
                        └────────┬────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
┌────────┴────────┐   ┌──────────┴──────────┐   ┌───────┴─────────┐
│  dim_product    │   │     fact_sales       │   │  dim_region     │
│─────────────────│   │─────────────────────│   │─────────────────│
│ PK: product_id  │───│ FK: date_id         │───│ PK: region_id   │
│ Product_Type    │   │ FK: product_id      │   │ Country         │
│ Brand           │   │ FK: region_id       │   └─────────────────┘
└─────────────────┘   │ FK: channel_id      │
                       │ FK: payment_id      │
┌─────────────────┐   │ Revenue_USD (metric)│   ┌─────────────────┐
│  dim_payment    │   │ Units_Sold (metric) │   │  dim_channel    │
│─────────────────│   └─────────────────────│   │─────────────────│
│ PK: payment_id  │───┘                     └───│ PK: channel_id  │
│ Payment_Method  │                             │ Sales_Channel   │
└─────────────────┘                             └─────────────────┘
```

### 📋 Deskripsi Setiap Tabel

**🔵 Fact Table: `fact_sales`**

Tabel inti yang menyimpan metrik penjualan dan foreign key ke semua dimensi.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `date_id` | INTEGER (FK) | Referensi ke dim_date |
| `product_id` | INTEGER (FK) | Referensi ke dim_product |
| `region_id` | INTEGER (FK) | Referensi ke dim_region |
| `channel_id` | INTEGER (FK) | Referensi ke dim_channel |
| `payment_id` | INTEGER (FK) | Referensi ke dim_payment |
| `Revenue_USD` | FLOAT | Total pendapatan per transaksi |
| `Units_Sold` | INTEGER | Jumlah unit terjual |

**🟡 Dimension Tables:**

| Tabel | Primary Key | Atribut |
|-------|-------------|---------|
| `dim_date` | `date_id` | Date, Day, Month, Year |
| `dim_product` | `product_id` | Product_Type, Brand |
| `dim_region` | `region_id` | Country |
| `dim_channel` | `channel_id` | Sales_Channel |
| `dim_payment` | `payment_id` | Payment_Method |

---

## 📁 Struktur Folder Proyek

```
📦 chocolate-sales-bi/
├── 📂 data/
│   ├── 📂 raw/
│   │   └── 📄 chocolate_sales_2025_dataset.csv   ← Dataset mentah asli
│   └── 📂 processed/
│       ├── 📄 dim_date.csv
│       ├── 📄 dim_product.csv
│       ├── 📄 dim_region.csv
│       ├── 📄 dim_channel.csv
│       ├── 📄 dim_payment.csv
│       └── 📄 fact_sales.csv
│
├── 📂 database/
│   └── 🗄️ chocolate_dw.db                        ← SQLite Data Warehouse
│
├── 📂 notebooks/
│   └── 📓 Chocolate_Collab.ipynb                 ← Main notebook (ETL + Analisis)
│
├── 📂 docs/
│   └── 📄 star_schema_diagram.png                ← Diagram ERD Star Schema
│
└── 📄 README.md                                  ← Dokumentasi proyek ini
```

---

## 📊 Key Insights

Berikut insight utama yang berhasil ditemukan dari analisis data:

### 🥇 Produk Terlaris (Berdasarkan Revenue)

| Rank | Produk | Total Revenue |
|------|--------|---------------|
| 🥇 1 | Chocolate Box | $151,378.86 |
| 🥈 2 | Milk Chocolate | $119,985.58 |
| 🥉 3 | Truffles | $116,117.81 |
| 4 | Chocolate Bar | $114,664.97 |
| 5 | Dark Chocolate | $108,655.80 |

> 💡 **Insight:** Chocolate Box menjadi pemimpin pasar dengan selisih signifikan, mengindikasikan preferensi konsumen pada produk bundling/gift untuk kebutuhan hadiah.

---

### 🌍 Pasar Terkuat per Negara

| Rank | Negara | Total Revenue |
|------|--------|---------------|
| 🥇 1 | France | $89,956.88 |
| 🥈 2 | Canada | $80,937.24 |
| 🥉 3 | India | $77,033.35 |
| 4 | UK | $76,796.23 |
| 5 | Germany | $74,267.03 |

> 💡 **Insight:** Prancis mendominasi pasar, mencerminkan budaya konsumsi cokelat premium yang kuat di Eropa. Munculnya India di posisi ke-3 menunjukkan potensi pasar Asia yang sedang berkembang.

---

### 🛒 Saluran Penjualan Paling Efektif

| Saluran | Total Revenue | Persentase |
|---------|---------------|------------|
| Convenience Store | $200,001.57 | 27.9% |
| Specialty Store | $180,633.82 | 25.2% |
| Supermarket | $179,683.38 | 25.1% |
| Online | $156,376.80 | 21.8% |

> 💡 **Insight:** Convenience Store menjadi kanal terkuat — konsumen masih lebih memilih pembelian impulsif di toko fisik untuk produk cokelat. Saluran online masih punya ruang pertumbuhan besar.

---

### 💳 Metode Pembayaran Favorit

| Metode | Total Revenue | Dominasi |
|--------|---------------|----------|
| 💻 Digital Wallet | $248,348.48 | 34.7% |
| 💵 Cash | $244,688.26 | 34.1% |
| 💳 Card | $223,658.83 | 31.2% |

> 💡 **Insight:** Digital Wallet menggeser cash sebagai metode pembayaran utama, menandakan percepatan adopsi pembayaran digital secara global bahkan untuk pembelian sehari-hari.

---

### 🏆 Brand Terkuat

| Rank | Brand | Total Revenue |
|------|-------|---------------|
| 🥇 1 | Toblerone | $118,291.29 |
| 🥈 2 | Mars | $115,337.39 |
| 🥉 3 | Lindt | $106,042.48 |
| 4 | Nestle | $100,439.80 |
| 5 | Ferrero | $93,826.16 |

> 💡 **Insight:** Toblerone memimpin tipis atas Mars — keduanya bersaing ketat. Lindt sebagai brand premium mampu bertahan di posisi ke-3 meski harga lebih tinggi.

---

### 📈 Ringkasan Keseluruhan

```
💰 Total Revenue     :  $716,695.57
📦 Total Units Sold  :  52,469 unit
📅 Periode Data      :  Jan 2025 — Des 2025
🌍 Cakupan Negara    :  10 negara
🏷️ Jumlah Brand      :  7 brand
🍫 Jenis Produk      :  6 kategori
```

---

## 🔗 Link Penting

| Platform | Link |
|----------|------|
| 📓 **Google Colab** | [Buka Notebook →](https://colab.research.google.com/) |
| 🐙 **GitHub Repository** | [Lihat Repository →](https://github.com/) |
| 📊 **Dataset (Kaggle)** | [Download Dataset →](https://www.kaggle.com/) |

