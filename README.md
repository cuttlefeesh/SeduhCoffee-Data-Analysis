# Seduh Coffee: End-to-End Data Analysis

Analisis data end-to-end untuk brand kopi spesialti *direct-to-consumer* (D2C) **Seduh**, mulai dari pembersihan data, pivot table, EDA, hingga segmentasi pelanggan RFM, dengan hasil akhir berupa rekomendasi berbasis data untuk mencapai target revenue 2026.

Proyek akhir **Data Analyst Bootcamp Talentiv** (Sept 2026).

> **Disclaimer:** Seluruh data bersifat fiktif dan tidak merepresentasikan bisnis atau merek tertentu. Proyek ini disusun semata-mata untuk tujuan pembelajaran.

---

## 1. Konteks Bisnis

**Seduh** adalah brand kopi spesialti D2C asal Indonesia (berdiri 2022). Produknya meliputi biji kopi single-origin, kopi siap minum (RTD), alat seduh, aksesori, paket langganan, dan gift set. Penjualan dilakukan lewat **Webstore, Tokopedia, Shopee, dan TikTok Shop**. Pelanggan utamanya adalah profesional muda perkotaan dan pecinta home brewing berusia 19–45 tahun. Pertumbuhan didukung iklan Instagram dan TikTok, program referral, serta konten organik.

**Situasi.** Menurut brief, revenue 2025 sekitar Rp4,4 M. Investor menetapkan **target Rp5,5 M pada 2026 (+24% YoY)** sambil meningkatkan retensi pelanggan dan efisiensi marketing.

**Masalah yang dihadapi manajemen:**
1. Biaya akuisisi pelanggan (CAC) naik lebih cepat daripada revenue.
2. Repeat purchase lemah: banyak pelanggan membeli sekali lalu tidak kembali.
3. Belum jelas produk, channel, bulan, atau segmen pelanggan mana yang benar-benar mendorong **profit**, bukan sekadar revenue.

**Tujuan analisis:** mengubah data operasional mentah menjadi diagnosis yang jelas dan rencana aksi untuk mencapai target 2026.

---

## 2. Dataset

| Tabel | Isi | Ukuran (mentah) |
|---|---|---|
| Orders | Transaksi: tanggal, produk, qty, harga, diskon, channel, kota, status, rating | ±25.800 baris |
| Customers | Profil: gender, usia, kota, tanggal daftar, channel akuisisi | 15.000 pelanggan |
| Products | Nama, kategori, harga, `unit_cost` | 50 produk |
| Marketing_Spend | Biaya iklan per bulan per channel | 96 baris |

Periode data: **Januari 2024 – Desember 2025**. Relasi tabel: `Orders.customer_id → Customers`, `Orders.product_id → Products`, dan `Marketing_Spend` di-join lewat bulan + channel.

---

## 3. Alur Pengolahan & Analisis Data

```
Problem Definition → Data Cleaning → Pivot Table → SQL → EDA → Visualisasi → Segmentasi RFM → Storytelling & Rekomendasi
```

### 3.1 Data Cleaning

| Masalah | Solusi | Teknik |
|---|---|---|
| Nilai kosong pada kolom `age` | Imputasi dengan median | `IF` + `MEDIAN` |
| Order duplikat | 14 duplikat dihapus | Remove Duplicates |
| Tipe data tidak konsisten (qty/tanggal sebagai teks, harga sebagai angka mentah) | Standarisasi format kolom | Format Cells (custom currency & date) |
| Typo, spasi berlebih, kapitalisasi tidak konsisten (channel, gender, kota) | Koreksi dan standarisasi, mis. *Shoppee → Shopee*, *Tokopdia → Tokopedia*, *F/Female → Perempuan* | `SUBSTITUTE`, `PROPER`, `TRIM` + tabel LookUp Mapping |
| Quantity tidak valid (0) | Diganti median quantity per produk | Tabel LookUp |
| Belum ada metrik finansial | Membuat kolom `revenue`, `unit_cost`, `profit`, `is_diskon`, `diskon value`, `tahun`, `bulan` | Formula aritmetika, `VLOOKUP` |

### 3.2 Definisi Metrik

- **Revenue** = `quantity × unit_price × (1 − discount_pct)`
- **Profit** = `quantity × (unit_price × (1 − discount_pct) − unit_cost)`
- Hanya order berstatus **Completed** yang dihitung. Order *Returned* dan *Cancelled* dikecualikan.
- **CAC** = total marketing spend ÷ jumlah pelanggan yang didapat dari channel tersebut
- **ROI marketing** = profit ÷ marketing spend
- **Repeat buyer** = pelanggan dengan lebih dari 1 order Completed
- **RFM**: Recency, Frequency, Monetary per pelanggan, dengan tanggal snapshot **1 Januari 2026**

---

## 4. Apa yang Dianalisis

| # | Pertanyaan Bisnis | Metode |
|---|---|---|
| Q1 | Kategori dan produk mana yang paling **profitable**? | Pivot Table (revenue, profit, margin per kategori/produk) |
| Q2 | Channel penjualan mana yang paling bernilai setelah diskon? | Pivot Table (profit dan rata-rata diskon per channel) |
| Q3 | Bagaimana tren revenue bulanan dan pola musiman (Ramadan, Harbolnas)? | Analisis tren + *lift vs baseline* |
| Q4 | Berapa proporsi one-time vs repeat buyer? | Pivot Table (nilai per pelanggan) |
| Q5 | Segmen pelanggan apa saja yang ada dan seberapa bernilai? | Segmentasi RFM (Python) |
| Q6 | Channel akuisisi mana yang paling bernilai dan efisien? | CAC dan ROI per channel |
| Q7 | Apakah diskon benar-benar menaikkan profit? | Analisis diskon vs unit/order vs profit/order |
| EDA | Bagaimana sebaran nilai order dan adakah outlier? | Statistik deskriptif, skewness, IQR |

---

## 5. Hasil & Insight

### EDA: Sebaran Nilai Order
- Nilai order condong ke kanan (**skewness 3,74**). Median Rp213.000, sedangkan rata-rata Rp282.627 (32,7% lebih tinggi karena tertarik order besar). Median quantity hanya 2 unit.
- Dengan metode IQR (batas atas Rp691.500) ditemukan **1.480 order outlier (6,17%)**. Outlier ini menyumbang **22,7% revenue dan 20,5% profit**, sehingga **tidak dihapus**.

### Q1. Produk Paling Profitable
- **Roasted Beans** adalah mesin profit utama: revenue Rp3,28 M, profit **Rp1,84 M (±56% dari total profit)**, margin **56,2%**. Produk andalannya adalah House Blend (dua varian menyumbang ±Rp508 jt profit dengan margin ±59%).
- **Brewing Equipment** berrevenue besar (±Rp840 jt) tetapi **margin terendah (29,1%)**.
- Margin keseluruhan: 48,2% dari total revenue Rp6,78 M dan profit Rp3,27 M.

### Q2. Channel Penjualan
- **Tokopedia** menghasilkan profit terbesar (Rp947 jt), tetapi **Webstore paling efisien**: diskon rata-rata hanya **3,26%** vs 5,3–6,4% di marketplace, dengan margin tertinggi dan AOV setara.
- **TikTok Shop** paling boros dalam diskon dengan margin terendah.

### Q3. Tren & Musiman
- Revenue cenderung naik sepanjang 2024–2025, dengan puncak pada Nov 2025 (Rp419 jt) dan Des 2025 (Rp390 jt).
- Dibanding baseline (rata-rata Rp262 jt/bulan): **Ramadan +12%**, sedangkan **Harbolnas Nov–Des +33–35%**, atau sekitar **3x lebih besar** daripada Ramadan.

### Q4. One-Time vs Repeat Buyer
- **62,4%** pelanggan hanya membeli sekali dan **37,6%** adalah repeat buyer.
- Repeat buyer menyumbang **65,4% revenue**, dan nilainya **3,13x** lebih besar per pelanggan (revenue Rp880 rb vs Rp282 rb; profit Rp424 rb vs Rp136 rb).

### Q5. Segmentasi RFM

| Segmen | % Pelanggan | % Revenue |
|---|---|---|
| Hibernating | 33,7% | 21,5% |
| Champions | 6,8% | 20,0% |
| Potential Loyalist | 29,4% | 18,9% |
| Loyal Customers | 9,2% | 17,6% |
| Need Attention | 14,6% | 10,3% |
| Can't Lose Them | 2,8% | 6,6% |
| At Risk | 3,5% | 5,2% |

- Sekitar **63%** pelanggan (Hibernating + Potential Loyalist) baru bertransaksi sekali. Inilah akar masalah "beli sekali lalu hilang".
- **Champions + Loyal** hanya 16% pelanggan tetapi menyumbang **37,6% revenue**.
- **At Risk + Can't Lose Them** (843 pelanggan) menyimpan **Rp861,8 jt (11,8%)** revenue yang berisiko hilang.
- Konversi Potential Loyalist ke frekuensi 2x berpotensi menambah ±Rp852,8 jt. Bersama revenue at risk, total potensi retensi ±**Rp1,71 M**.

### Q6. Channel Akuisisi

| Channel | CAC | ROI |
|---|---|---|
| Google Search | Rp58.319 | **4,75x** |
| Instagram Ads | Rp103.633 | 1,93x |
| Marketplace Ads | Rp91.351 | 1,54x |
| TikTok Ads | Rp123.983 | **1,03x** |

- **Google Search 4,6x lebih efisien** daripada TikTok Ads. TikTok Ads punya CAC tertinggi, nilai pelanggan terendah, dan ROI nyaris impas.
- **Referral** (tanpa biaya iklan) menghasilkan revenue per pelanggan tertinggi (±Rp690 rb) dari seluruh channel.

### Q7. Diskon vs Volume vs Profit

| Diskon | Unit/order | Profit/order |
|---|---|---|
| 0% (baseline) | 2,17 | Rp135.170 |
| 5% | 2,53 | **Rp146.050** |
| 20% | 3,28 | Rp123.467 |
| 25% | 3,55 | Rp115.734 |

- Diskon menaikkan volume, tetapi tidak linear. **Profit/order memuncak di diskon 5%**, dan pada diskon ≥20% profit turun **di bawah baseline tanpa diskon** walau unit terjual lebih banyak.

---

## 6. Rekomendasi

| Area | Rekomendasi |
|---|---|
| **Produk** | Dorong Roasted Beans lebih agresif lewat bundling dan cross-sell. Jadikan Brewing Equipment produk penarik pelanggan (bukan andalan profit). |
| **Channel** | Migrasikan pelanggan marketplace ke Webstore lewat insentif first-order. Tinjau ulang strategi diskon di TikTok Shop. |
| **Musiman** | Alokasikan budget marketing dan stok terbesar pada **Okt–Des** (persiapan sebelum Harbolnas), bukan disebar rata sepanjang tahun. |
| **Retensi** | Buat program retensi terjadwal (follow-up WA/email pasca pembelian pertama, kupon repeat order) untuk one-time buyer. Dorong Subscription agar repeat purchase terjadi otomatis. |
| **Segmentasi** | Champions/Loyal: program VIP/early access. Hibernating: reaktivasi lewat channel murah. Potential Loyalist: kupon terjadwal. At Risk/Can't Lose Them: win-back personal. |
| **Akuisisi** | Realokasikan budget iklan TikTok ke Google Search dan bangun program referral. |
| **Promo** | Batasi diskon maksimal **10%**. Hindari diskon ≥20% kecuali untuk clearance stock. |

**Kesimpulan:** potensi dari retensi (win-back + konversi Potential Loyalist) berukuran sebanding dengan tambahan revenue yang dibutuhkan, sehingga target 2026 dapat dikejar dengan lebih mengandalkan pelanggan yang sudah ada dan lebih sedikit pada akuisisi baru yang mahal.

---

## 7. Tools

- **Microsoft Excel**: Pivot Table, `VLOOKUP`, `IF`/`MEDIAN`, `SUBSTITUTE`/`PROPER`/`TRIM`, chart
- **SQL**: pengambilan dan penggabungan data
- **Python (Pandas)**: segmentasi RFM dan visualisasi
- **Data Storytelling**: deck untuk stakeholder

## 8. Struktur Workbook Excel

| Sheet | Isi |
|---|---|
| Project Brief, Data Dictionary | Brief kasus dan definisi kolom |
| Orders, Customers, Products, Marketing_Spend | Data mentah |
| Order-Master, Customer-Master, Products-Master | Data bersih + kolom hasil feature engineering |
| LookUp Mapping | Tabel standarisasi channel, gender, dan quantity |
| EDA | Statistik deskriptif dan deteksi outlier |
| Q1-Q3, Q4-Q7 | Pivot table dan chart per pertanyaan bisnis |

## 9. Keterbatasan

- Data bersifat fiktif sehingga hasil tidak dapat digeneralisasi ke bisnis nyata.
- Analisis bersifat deskriptif dan korelasional. Estimasi potensi revenue (mis. Rp852,8 jt) bergantung pada asumsi, seperti target frekuensi pembelian 2x.

---

**Author:** Chandra Aulia Haswangga: Data Analyst Enthusiast, S1 Teknik Komputer, Telkom University
