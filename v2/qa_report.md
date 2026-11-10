# QA Report — Tokopedia v2

Dibuat otomatis oleh `scripts/assemble_v2.py`.

Pengecualian pengguna: **kolom konstan diperbolehkan** (mis. `sort`, `crawler_version`, `is_ad`). Kolom kosong tetap harus diperiksa.

| Pemeriksaan | Hasil | Status |
|---|---|---|
| Baris ≥ 1400 | 1558 | LULUS |
| product_id unik 100% | 1558/1558 | LULUS |
| Tidak ada kolom kosong | tidak ada | LULUS |
| Kolom konstan (diizinkan pengguna) | ['sort', 'page_number', 'is_ad', 'crawler_version'] | n/a |
| `is_ad` konstan `false` | endpoint pencarian tidak mengekspos flag iklan; brief: tandai *jika* terdeteksi | n/a |
| Non-resmi ≥ 600 (~40%) | 781 (50.1%) | LULUS |
| URL berawalan tokopedia.com | 1558/1558 | LULUS |
| price > 0 | 1558/1558 | LULUS |
| rating 0–5 atau kosong | 1558/1558 | LULUS |
| sold ≥ 0 | 1558/1558 | LULUS |
| sold ≥ batas sold_label | 1555/1555 | LULUS |
| Produk tanpa rating dilaporkan | 2 | n/a |
| crawl_timestamp terisi | 2026-11-05T18:56:30.538657+07:00 → 2026-11-05T19:01:57.420282+07:00 | LULUS |
| price terisi ≥99% | 100.0% | LULUS |
| sold terisi ≥99% | 100.0% | LULUS |
| review_count terisi ≥99% | 100.0% | LULUS |
| rating terisi ≥99% | 99.9% | LULUS |

## Median vs v1

| Fitur | v2 | v1 |
|---|---|---|
| price | 99,974 | 208,000 |
| sold | 13 | 17 |
| review_count | 3 | 4 |

## Catatan keterbatasan

- `category`, `category_breadcrumb`, `category_id` terisi 87,4%. Sisanya (196 baris) memang tidak dikembalikan oleh endpoint pencarian maupun PDP; tidak diisi dengan tebakan.
- `original_price_fmt`/`original_price` terisi 59,3% karena hanya produk berdiskon yang punya harga asli; `discount_pct` = 0 untuk sisanya.
- `is_ad` konstan `false`: endpoint `ace_search_product_v4` tidak mengekspos penanda iklan.
- `crawl_timestamp` adalah waktu pengambilan pada fase pencarian.
- **Skew rating**: 97,5% produk tepat 5,0 (hanya 7 nilai berbeda, 3,0–5,0). Sama seperti v1 (~91% di 5,0) — ini perilaku sumber PDP Tokopedia, bukan bug parsing. Variasi rating rendah, catat di pembahasan KTI.
## Komposisi per kata kunci

| Kata kunci | Resmi | Non-resmi |
|---|---|---|
| ac 1 pk | 15 | 20 |
| blender | 20 | 20 |
| cokelat bubuk | 19 | 19 |
| gamis | 19 | 20 |
| headphone bluetooth | 20 | 20 |
| iphone | 20 | 17 |
| jam tangan pria | 19 | 20 |
| kamera canon | 20 | 20 |
| kasur | 20 | 20 |
| kemeja pria | 20 | 18 |
| kopi bubuk | 19 | 19 |
| kulkas | 20 | 20 |
| kursi kantor | 20 | 20 |
| lampu led | 20 | 20 |
| laptop asus | 20 | 20 |
| laptop lenovo | 20 | 20 |
| lipstik | 20 | 20 |
| mainan anak | 20 | 19 |
| masker | 20 | 19 |
| meja belajar | 17 | 20 |
| mesin cuci | 20 | 20 |
| minyak goreng | 20 | 19 |
| panci | 19 | 20 |
| parfum | 19 | 20 |
| popok bayi | 17 | 19 |
| power bank | 20 | 20 |
| printer | 20 | 20 |
| raket badminton | 20 | 19 |
| rice cooker | 19 | 20 |
| samsung galaxy | 20 | 19 |
| sepatu running | 20 | 20 |
| sepeda | 20 | 19 |
| serum wajah | 20 | 20 |
| smart tv | 20 | 20 |
| susu formula | 20 | 19 |
| tas ransel | 19 | 20 |
| teh celup | 18 | 20 |
| vitamin c | 20 | 19 |
| xiaomi | 19 | 19 |
| yoga mat | 19 | 18 |
