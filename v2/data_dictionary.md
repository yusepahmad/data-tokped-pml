# Kamus Data — tokopedia_products_v2.csv

Sumber: pencarian relevance `ace_search_product_v4` (kolom pencarian) dan
halaman detail produk / PDP Apollo cache (kolom rating, review_count, sold).

| Kolom | Tipe | Satuan / nilai | Sumber | Keterangan |
|---|---|---|---|---|
| product_id | string | — | pencarian | ID produk Tokopedia, unik |
| name | string | — | pencarian | Nama produk |
| price | number | Rupiah | pencarian | Harga sekarang |
| rating | number | 0–5 | PDP (`basicInfo.stats.rating`) | Kosong bila produk belum berating |
| review_count | integer | jumlah | PDP (`basicInfo.stats.countReview`) | 0 bila belum ada ulasan |
| sold | number | unit | PDP (`basicInfo.txStats.countSold`) | Jumlah terjual aktual |
| sold_label | string | teks | pencarian (`labelGroups`) | Label bucket, mis. "100+ terjual" |
| category | string | — | pencarian (`categoryBreadcrumb`) | Kategori tingkat atas (segmen pertama) |
| category_breadcrumb | string | — | pencarian | Jejak kategori lengkap dipisah `/` |
| category_id | number/string | — | PDP (`basicInfo.category.id`) | ID kategori; fallback ke pencarian |
| url | string | — | pencarian | `https://www.tokopedia.com/…` |
| shop_name | string | — | pencarian (`shop.name`) | Hanya untuk internal |
| shop_city | string | — | pencarian (`shop.city`) | Kosong bila tidak ada |
| is_official | boolean | — | pencarian (`shop.isOfficial`) | Toko resmi vs bukan |
| source_keyword | string | — | — | Kata kunci pertama produk ditemukan |
| sort | string | "relevance" | — | Selalu relevance (konstan) |
| discount_pct | number | % | dihitung | `(1 − price/original_price) × 100` |
| original_price_fmt | string | teks | pencarian (`originalPrice`) | Harga asli terformat |
| original_price | number | Rupiah | dihitung dari `original_price_fmt` | |
| crawl_timestamp | string | ISO 8601 WIB | — | Saat produk diambil dari pencarian |
| page_number | integer | 1–N | — | Halaman hasil relevansi |
| rank_in_page | integer | 1–60 | — | Posisi pada halaman |
| shop_id | string | — | pencarian (`shop.shopId`) | Stabil per toko |
| shop_tier | integer | 2/3 | filter `shop_tier` | 2=Toko Resmi, 3=Gold Merchant (non-resmi); divalidasi terhadap `goldOS.shopTier` |
| is_ad | boolean | false | — | Tidak terdeteksi pada endpoint ini (konstan false) |
| n_keyword_hits | integer | jumlah | dihitung | Jumlah kata kunci yang memunculkan produk |
| crawler_version | string | v2.0.0 | — | Versi skrip crawler |

## Catatan penting

- **Kuota toko resmi/non-resmi** dipenuhi lewat filter pencarian `shop_tier`:
  `shop_tier=2` (Toko Resmi) dan `shop_tier=1` (Gold Merchant, non-resmi), keduanya
  dengan urutan relevance (`ob=23`). Tanpa filter, urutan relevance hanya
  memunculkan Toko Resmi (terverifikasi: 100% pada sampel 1.080 produk).
- **Tingkat non-resmi yang terambil** adalah Gold Merchant (toko terverifikasi
  emas, `is_official=false`). Toko reguler (non-gold, non-resmi) tidak muncul di
  urutan relevance, hanya di urutan `latest` yang dilarang brief.
- **Skew rating**: 97,5% baris tepat 5,0 (hanya 7 nilai: 3,0–5,0). Ini sifat
  sumber PDP, sama seperti v1 (~91%). `rating` valid tetapi variansnya rendah;
  pertimbangkan transformasi/binarisasi bila dipakai untuk clustering.
- **Kategori kosong pada 196 baris**: endpoint pencarian tidak mengembalikan
  `categoryBreadcrumb` untuk produk tersebut; tidak diisi dengan tebakan.
- **Kolom kosong/konstan di v1 diperbaiki**: `discount_pct` dihitung ulang,
  `tx_success`, `view_count`, `total_review`, `price_pdp` dibuang (selalu
  kosong/0 di PDP), `total_rating` dibuang karena duplikat `review_count`.
