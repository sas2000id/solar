# Solar PV Investment & Savings Simulator v1.1

Aplikasi web statis interaktif untuk simulasi awal investasi PLTS atap industri/pabrik.

## Fitur
- Rekomendasi awal kapasitas PV dari konsumsi tahunan (kWh aktual jika diimpor; jika tidak, estimasi tagihan ÷ tarif).
- Parameter tarif, target offset, specific yield, self-consumption, biaya EPC, O&M, periode, discount rate, degradasi, eskalasi tarif, dan kapasitas manual.
- Metrik: CAPEX, penghematan tahun pertama, simple payback, NPV, IRR, ROI kumulatif, produksi tahun pertama.
- Grafik arus kas kumulatif dan penghematan bersih tahunan.
- Tabel arus kas tahunan, validasi IRR berbasis bisection + residual NPV, ekspor CSV, cetak/simpan PDF.
- Impor tagihan PLN dari CSV/XLS/XLSX (Excel membutuhkan koneksi internet untuk memuat SheetJS dari CDN).
- Template CSV dan panduan deployment Vercel.

## Menjalankan lokal
1. Ekstrak ZIP.
2. Buka `index.html` di browser modern.
3. Untuk penggunaan yang lebih konsisten, jalankan server lokal dari folder ini:
   - Python: `python -m http.server 8000`
   - Buka `http://localhost:8000`
4. Grafik menggunakan Chart.js CDN dan impor Excel menggunakan SheetJS CDN; kedua fitur tersebut memerlukan internet. Impor CSV dan kalkulasi inti berjalan di browser.

## Format file tagihan
Header baris pertama, contoh:
```csv
Bulan,Tagihan_Rp,Konsumsi_kWh
2026-01,98000000,65000
2026-02,101000000,67000
2026-03,99500000,66000
```
Kolom yang dikenali mencakup variasi nama `bulan/tanggal/periode`, `tagihan/amount/total/rupiah/billing/Rp`, dan `kWh/konsumsi/consumption/energy/pemakaian`. Angka boleh berupa `98000000` atau `98.000.000`; pastikan pemisah angka tidak ambigu. Data dianggap per periode/baris dan rata-rata baris valid dihitung sebagai nilai rata-rata. Jangan campurkan satuan atau periode yang berbeda.

## Deploy ke Vercel
### Opsi A — melalui dashboard Vercel
1. Buat repository GitHub baru, misalnya `solar-pv-investment-simulator`.
2. Unggah `index.html` dan `README.md` ke root repository.
3. Buka Vercel dan pilih **Add New → Project** lalu impor repository tersebut.
4. Framework preset: **Other**. Build command dan output directory tidak diperlukan untuk aplikasi statis; jika diminta, kosongkan build command dan gunakan `.` sebagai output directory.
5. Klik **Deploy**. Setelah deploy, uji slider, grafik, template CSV, impor CSV, dan ekspor arus kas.

### Opsi B — Vercel CLI
1. Instal Node.js LTS dan Vercel CLI: `npm install -g vercel`.
2. Ekstrak ZIP, lalu masuk ke folder proyek.
3. Jalankan `vercel` dan ikuti instruksi. Untuk rilis produksi jalankan `vercel --prod`.
4. Jika Vercel meminta pengaturan build, pilih proyek statis tanpa build step.

Tidak ada backend/database dalam versi ini; data yang diimpor tidak dikirim ke server. Ini adalah aplikasi browser-side. Jangan unggah data yang bersifat rahasia jika organisasi Anda tidak mengizinkannya.

## Cara kerja perhitungan
- Konsumsi tahunan = rata-rata kWh bulanan × 12. Jika tidak ada kWh impor, estimasi konsumsi = rata-rata tagihan bulanan ÷ tarif × 12.
- Kapasitas rekomendasi = konsumsi tahunan × target offset ÷ specific yield, dibulatkan ke atas ke kelipatan 5 kWp.
- Produksi tahun 1 = kapasitas × specific yield.
- Hemat kotor tahun t = produksi tahun t × self-consumption × tarif × eskalasi tarif.
- O&M tahunan = CAPEX × persentase O&M.
- Arus kas tahun 0 = −CAPEX; tahun berikutnya = penghematan kotor − O&M.
- NPV = jumlah arus kas terdiskonto pada discount rate.
- IRR = akar NPV(r)=0, dicari dengan metode bisection. Hasil hanya ditampilkan jika arus kas menunjukkan pola konvensional dan residual NPV setelah penyelesaian lolos toleransi.
- ROI kumulatif = (total arus kas operasi selama periode − CAPEX) ÷ CAPEX × 100%.

## Batasan penting
Model ini belum menghitung pajak, biaya pembiayaan, biaya penggantian inverter, nilai sisa, penurunan/peningkatan beban, biaya interkoneksi, demand charge, ekspor listrik, biaya izin, atau profil beban per jam. Tarif efektif dan specific yield perlu diverifikasi. Untuk keputusan investasi final, lakukan audit energi, kajian struktur/kelistrikan, studi interkoneksi, dan penawaran EPC terperinci.
