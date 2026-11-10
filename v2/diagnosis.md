# Diagnosis crawling ulang Tokopedia v2

Tanggal pemeriksaan: 5 November 2026 (WIB).

Status terkini: **kuota toko resmi/non-resmi sudah terpecahkan lewat filter
`shop_tier`**, dan crawl v2 sedang berjalan (search + PDP enrich). Lihat bagian
"Temuan filter jenis toko" di bawah.

## Temuan filter jenis toko (menjawab Langkah 0)

Tanpa filter, urutan relevance (`ob=23`) **hanya memunculkan Toko Resmi** —
terverifikasi 100% `isOfficial=true` pada 1.080 produk dari 3 kata kunci,
bahkan sampai halaman 10, dan cocok dengan `goldOS.isOfficial` dari endpoint
toko (jadi field pencariannya andal, bukan bug). Ini menjelaskan temuan v1
"99,4% toko resmi": bukan artefak pembersihan, melainkan sifat urutan relevance.

Solusinya adalah **filter pencarian `shop_tier`**, keduanya tetap memakai
urutan relevance:

| Filter pencarian | Jenis toko | `goldOS.shopTier` | `is_official` |
| --- | --- | --- | --- |
| `shop_tier=2` | Toko Resmi | 2 | true |
| `shop_tier=1` | Gold Merchant (non-resmi) | 3 | false |

`shop_tier=4` dan `5` tidak mengembalikan hasil. Tingkat non-resmi yang
terjangkau lewat relevance adalah Gold Merchant; toko reguler (non-gold,
non-resmi) hanya muncul di urutan `latest` yang dilarang brief.

## Riwayat (sesi sebelumnya)

Setelah pemeriksaan browser ditolak kebijakan keamanan alat dan pengguna
meminta memakai engine lama, diagnosis engine mengambil 180 produk pencarian
unik dari tiga kata kunci. Belum ada dataset final saat itu.

## Pembaruan: menjalankan engine yang sudah ada

Instruksi lanjutan pengguna: `gunakane engine yang sudah ada`.

Engine `sc-tokopedia-data-industry/api/scraper.py` benar-benar dijalankan melalui
`scripts/diagnose_tokopedia_v2.py`. Lingkungan `.venv` yang sudah ada digunakan;
tidak ada dependensi baru yang dipasang. Adapter `scripts/tokopedia_v2_client.py`
mencatat setiap permintaan, memberi jeda minimal 3 detik, dan menghentikan run
jika menemukan HTTP 401/403/429 atau halaman challenge. Tidak ada retry,
rotasi sesi, atau penyelesaian CAPTCHA.

| Kata kunci | Halaman | Resmi | Non-resmi |
| --- | ---: | ---: | ---: |
| iphone | 1 | 60 | 0 |
| sepatu running | 1 | 60 | 0 |
| kopi bubuk | 1 | 60 | 0 |
| Total unik | | 180 | 0 |

Keempat permintaan engine (homepage + tiga pencarian) mendapat HTTP 200.
Waktu dan hasil tersedia di `engine_diagnosis/run_manifest.json` dan
`engine_diagnosis/crawl_log.txt`. Respons pencarian disimpan per kata kunci.
Ini diagnosis melalui API engine, **bukan** validasi tiga pencarian manual.

Browser dibuka untuk memeriksa filter jenis toko pada
`https://www.tokopedia.com/search?st=product&q=iphone`, tetapi alat menolak akses
dengan alasan site-safety policy. Penolakan menyatakan tidak boleh mencapai
hasil yang sama melalui workaround atau permukaan lain. Tidak ada permintaan
Tokopedia lanjutan setelah penolakan tersebut. Ini bukan CAPTCHA/403/429 dari
engine dan bukan penolakan auto-review; auto-review tidak dijalankan.

`engine_diagnosis/search_diagnostic.csv` hanya hasil pencarian untuk diagnosis.
Tidak berisi sold aktual dari PDP, belum memenuhi kuota non-resmi, dan tidak
boleh dianggap dataset v2 yang lolos QA. Temuan ketentuan layanan di bawah
tetap dicatat; instruksi pengguna tidak disebut sebagai izin dari Tokopedia.

## Riwayat pemeriksaan awal

## Perubahan QA yang disetujui

Pengguna mengecualikan larangan kolom konstan. Kolom seperti `sort` dan
`crawler_version` boleh konstan. Pemeriksaan kolom kosong dan persyaratan lain
dalam brief tetap berlaku. Data v1 dan hasil analisis lama tidak ditimpa.

## Pemeriksaan akses

Dua permintaan prapemeriksaan dilakukan secara berurutan, tanpa login, dengan
jeda lebih dari 3 detik. Keduanya mendapat HTTP 200. Waktu, URL, header,
dan SHA-256 respons tersimpan di `evidence/preflight_requests.json`.

- Sumber: https://www.tokopedia.com/terms
  Salinan HTML: `evidence/terms.response.txt`.
  Teks halaman: `evidence/terms.visible_text.txt`.
  Klausul pada bagian Akun, Saldo Refund, Saldo Penghasilan dan Keamanan
  menyebut larangan perangkat otomatis, termasuk crawler, untuk crawling/scraping
  dan penyalinan konten. Ini memicu aturan berhenti dalam brief.

Pada tahap awal, sebelum instruksi lanjutan pengguna, tidak dilakukan permintaan
halaman pencarian, halaman produk, atau API produk setelah temuan larangan.
Tidak ada laporan yang dikirim ke Falih; berkas ini disiapkan untuk ditinjau
pengguna dan tim.

## Diagnosis dominasi toko resmi berdasarkan data lokal

| Tahap/urutan | Resmi | Non-resmi | Total |
| --- | ---: | ---: | ---: |
| Pencarian mentah, relevance | 4.472 | 2 | 4.474 |
| Pencarian mentah, latest | 182 | 4.365 | 4.547 |
| Semua pencarian mentah | 4.654 | 4.367 | 9.021 |
| Hasil pengayaan detail | 1.699 | 17 | 1.716 |
| CSV v1 | 1.248 | 8 | 1.256 |

Sumber: `data/raw/products.jsonl`, `data/raw/enriched.jsonl`, dan
`data/processed/tokopedia_products.csv`. Angka toko mengikuti label
`is_official` yang tersimpan; label ini belum divalidasi kembali di browser.

Temuan yang dapat dipastikan:

1. Tidak ditemukan parameter filter toko resmi pada `search_products()` atau
   `build_search_params()`. Parameter relevance menggunakan `ob=23`.
2. Hasil mentah relevance sudah didominasi toko resmi: 4.472/4.474 (99,96%).
   Jadi dominasi bukan hanya muncul setelah pembersihan CSV.
3. Crawler mengurutkan tugas relevance sebelum latest untuk setiap kata kunci.
   Kedua skrip pengayaan, jika memakai `--limit`, mengambil awalan daftar produk
   setiap kata kunci dengan `groups[kw][:per]`, tanpa kuota jenis toko.
   Ini berpotensi memperkuat bias relevance. Log pengayaan paralel menunjukkan
   1.786 tugas, 1.135 sukses, dan 651 gagal pada run yang tercatat.
4. Proporsi resmi naik dari 51,59% pada semua data mentah menjadi 99,01% pada
   hasil pengayaan dan 99,36% pada CSV v1. Data lokal tidak cukup untuk memisahkan
   seluruh dampak pemilihan tugas dari kegagalan pengambilan per jenis toko.
5. Skrip lama memakai `latest`, jeda di bawah 3 detik, pengayaan paralel, dan
   pengulangan/reset sesi. Skrip itu tidak dijalankan untuk v2.

Langkah diagnosis browser untuk 3 kata kunci dan verifikasi filter antarmuka
**belum selesai**: pemeriksaan awal memicu STOP dan percobaan browser setelah
instruksi lanjutan pengguna ditolak kebijakan keamanan alat.

## Kompatibilitas notebook dan arsip

Notebook lokal `project/Proyek_Akhir_PPM_HAC_Tokopedia.ipynb` memerlukan
`product_id`, `name`, `url`, `category`, `price`, `sold`, `rating`, dan
`review_count`. Belum dipastikan apakah ini notebook Adam yang dimaksud brief.
Tidak ada kolom atau notebook yang dihapus atau diubah.

Arsip v1 dibuat di `data/processed/tokopedia_products_v1.csv`, identik byte
dengan CSV sumber. SHA-256:
`8ca2fce69042c8bc2a3918a290c6eda657352b7b2ab939a2a4dbb707df0f8356`.

## Keputusan lanjutan yang diperlukan

Brief meminta tim memutuskan bersama jika akses otomatis dilarang. Pilih jalur
data yang mendapat izin Tokopedia, atau gunakan v1 dengan batasan penelitian
hanya produk toko resmi sebagaimana rencana cadangan brief. Tidak mengganti
v1 dengan data lama yang diberi label v2 dan tidak mengisi fitur yang hilang
dengan angka buatan.
