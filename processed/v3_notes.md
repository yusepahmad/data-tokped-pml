# Dataset v3 — Tokopedia Products (2026-11-10)

`tokopedia_products_v3.csv` — **23.209 baris × 27 kolom**, skema identik dengan v2
(silakan pakai langsung di notebook; hanya ganti nama file di `DATA_URL`).

## Apa yang berbeda dari v2 (1.558 baris)

| | v2 | v3 |
|---|---|---|
| Produk unik | 1.558 | 23.209 |
| Kedalaman koleksi | 1 halaman per keyword×tier | hingga 5 halaman |
| Keyword | 40 | 41 (inti sama) |
| Enrichment PDP | ~63% sukses | 99,7% sukses |
| Metode | sama: searchProductV5 + filter shop_tier (2=resmi, 1=gold) ×2 per keyword | sama |

## Karakteristik distribusi (penting untuk clustering)

- `price` / `sold` / `review_count` sangat right-skewed (skewness 10,2 / 39,6 / 34,7);
  `log1p` menurunkannya ke 0,7 / 1,2 / 1,7
- **86,3% produk rating tepat 5,0** (vs 97,5% di v2) — sampling lebih dalam memunculkan
  lebih banyak produk dengan banyak review (3.188 produk rating <5). Fenomena rating=5
  dominan tetap: rating = rata-rata bintang, median review hanya 3, ulasan marketplace
  Indonesia didominasi 5 bintang
- Campuran toko tetap seimbang ±50/50 resmi/gold merchant

## Provenance

- Tanggal crawl: 2026-11-10 (WIB), kolektor: TokpedMarketScrapper v2.1.0-catalog
  (github.com/muhfalihr/laptopmarketscrapper)
- Juga dipublikasikan di Kaggle: venussss/tokopedia-product-market-dataset (v2.1.0)
- Lisensi: CC-BY-SA-4.0, keperluan akademik nonkomersial
