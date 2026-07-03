<h1>Telkomsel Billing & Network Data Forensics: Mitigasi Kebocoran Pendapatan & Audit Integritas Sistem</h1>

## 1. Project Overview

Proyek ini merupakan analisis audit forensik data skala besar pada log transaksi internet Telkomsel (`usage_raw` & `user_registration`) dengan total objek audit sebanyak **300.000 sesi**. Fokus utama dari proyek ini adalah **mengidentifikasi celah kebocoran pendapatan (*revenue leakage*) lintas direktorat, mengevaluasi efektivitas subsidi program bundling premium, mendeteksi kerusakan integrasi sistem internal (*systemic bug*), serta memitigasi risiko kepatuhan regulasi**.

Key Objectives:

- Proteksi Anggaran Komersial: Mengevaluasi integritas program *Halo Bundling* premium dengan mendeteksi pola penyalahgunaan pemisahan kartu SIM dari perangkat subsidi (*IMEI Mismatch*).
- Audit Integrasi Gateway Data: Menyelidiki penyebab hilangnya data registrasi identitas (NIK/KK) pelanggan untuk memitigasi risiko hukum pemblokiran massal oleh regulator (Kominfo).
- Optimasi Infrastruktur & Pipeline: Mengidentifikasi kebocoran trafik sebelum aktivasi (*Provisioning Delay Bug*) serta memperbaiki fragmentasi format pencatatan log jaringan demi akurasi pelaporan manajemen.
- Validasi Sistem Billing: Memisahkan noise sistemik (*hardcoded error codes*) dari perilaku konsumsi riil manusia guna menetapkan batas kontrol operasional baru.

## 2. Data Sources

- tsel_data_usage.csv - Log mentah berisi 300.000 data transaksi sesi internet pelanggan, mencakup metrik volume pemakaian (`payload_mb`), tipe jaringan (`network_type`), ID sesi, tanggal, dan identitas perangkat (`used_imei`).
- tsel_subscribers.csv – Berisi dimensi pelanggan yang mencakup nomor hp pelanggan (`msidn`), tanggal kartu sim di aktivasi (`activation_date`),  status registerasi NIK/KK (`nik_kk_status`), dan registrasi IMEI martphone yang didaftarkan saat beli paket bundling ( Bisa NaN jika bukan paket bundling) (`registered_imei`)
- tsel_packages.csv - Berisi detail code paket (`package_code`), nama paket (`package_name`), dan harga untuk setiap paket (`price`)

## 3. Technologies Used

* **Programming Language:** Python 3.12 (Akurasi analisis dan jaminan kompatibilitas library data science)
* **Data Manipulation & Analysis:** Pandas, NumPy, Scikit-learn, SciPy (Modul statistik inferensial: T-Test, Chi-Square, ANOVA, Mann-Whitney U)
* **Data Visualization:** Matplotlib, Seaborn (Visualisasi distribusi curve, boxplot analisis pencilan, dan reporting)
* **Environment & Tools:** Homebrew, Virtual Environments (`venv`), Jupyter Notebook
* **Version Control:** Git

## 4. Project Structure

```
├── README.md                  <- Berkas dokumentasi utama proyek.
|
├── data
│   ├── raw                    <- Data log mentah dari core network gateway (GGSN) dan database registrasi.
│   └── cleaned                <- Data yang telah dibersihkan dari nilai dummy error 1 TB dan nilai negatif.
│
├── notebooks                  <- Jupyter notebooks untuk eksperimen.
│   └── ProgramSubsidiTelkomselDataExploration.ipynb
│
├── reports                    <- Dokumen analisis untuk jajaran eksekutif/C-Level.
│   ├── slide                  <- Slide presentasi PowerPoint (Impact-Driven, C-Level Ready).
│   └── figures                <- Grafik visualisasi hasil export plot untuk presentasi (mentah vs normal).
│
├── requirements.txt           <- Berkas dependensi pustaka untuk mereproduksi environment analisis.
│
└── src                        <- Kode sumber modular (skrip ETL pipeline untuk pembersihan otomatis).
```

## 5. Summary of Finding

### 5.1 Business Insight

* **Fraud Subsidi Bundling (Marketing Directorate Impact):**
  Ditemukan 17.183 sesi internet mengalami  *IMEI Mismatch* . Hasil *Independent T-Test* (**$T = -0.8520$**, **$P = 0.3942$**) membuktikan perilaku konsumsi data kelompok *Mismatch* adalah **homogen** (~1 GB s.d 2 GB) dengan pengguna normal. Oknum tidak melakukan abuse kuota massal, melainkan mengecer kartu premium ke pengguna prabayar biasa. Hal ini menyebabkan **kebocoran subsidi gawai sebesar Rp1.675.293.000** dan gagalnya akuisisi segmen pelanggan  *Ultra High-Value* .
* **Bug Integrasi Identitas & Risiko Hukum (Finance & Risk Impact):**
  Sebanyak 14.8% data registrasi NIK/KK kosong secara flat dan homogen di semua jenis paket, termasuk paket premium (Uji  *Chi-Square* , **$P = 0.7118$**). Ini merupakan bukti mutlak adanya **Bug Sistemik Internal** pada gateway integrasi dengan Dukcapil, bukan fraud eksternal. Dampaknya, jika dilakukan penegakan hukum oleh Kominfo, Telkomsel menghadapi **Revenue at Risk sebesar Rp4.117.179.000** dari pemblokiran massal pengguna aktif tersebut.
* **Provisioning Delay Bug & Fragmentasi Pipeline (Network & IT Impact):**
  Terdeteksi 12.008 sesi internet ilegal (4.00% dari total log) yang aktif **sebelum** tanggal aktivasi kartu karena latensi sinkronisasi temporal antara gateway jaringan (GGSN) dengan database billing pusat. Selain itu, **54.96% log penulisan tipe jaringan cacat format** (terfragmentasi menjadi 10 variasi penulisan seperti 4G, LTE, 4G LTE), menciptakan risiko tinggi *Misleading Reporting* pada evaluasi kapasitas jaringan.
* **Validasi Pembersihan Data Billing:**
  Nilai pemakaian maksimal sebesar `999.999,90 MB` (~1 TB) terbukti secara statistik ( *Mann-Whitney U Test* , **$P = 0.0$**) merupakan produk dari mekanisme *hardcoded error handling* dari mesin billing saat kalkulasi gagal, bukan akibat perilaku pemakaian riil pengguna. Setelah dibersihkan, rata-rata konsumsi normal manusia adalah **1.023,50 MB** per sesi dengan batas atas wajar tertinggi (Persentil 99.9) sebesar  **2.045,91 MB** .

### 5.2 Actionable Recommendation

* **Komersial & Proteksi Produk (Marketing):** Terapkan kebijakan *Strict Hard-Locking* IMEI secara otomatis pada jaringan. Jika kartu SIM dipasang di gawai yang tidak sesuai selama 3 sesi berturut-turut, matikan akses data secara otomatis. Ubah skema kuota besar di depan menjadi kuota suntikan bulanan berkala ( *recurring monthly quota* ).
* **Arsitektur Jaringan & Sinkronisasi Sistem (Network & IT):** Tingkatkan performa API sinkronisasi temporal untuk memotong latensi *provisioning* jaringan guna mengeliminasi 4% sesi ilegal sebelum aktivasi. Pasang *Data Transformation Layer* (fungsi standarisasi otomatis) pada pipa *ingestion data lakehouse* untuk menyatukan fragmentasi penulisan tipe jaringan yang rusak sebesar 54.96%.
* **Tata Kelola Sistem Billing (Data Governance):** Hentikan penulisan kode eror berbentuk angka *dummy* (seperti 1 TB atau nilai negatif) langsung di kolom pengukuran utama (`payload_mb`). Alihkan status gagal kalkulasi tersebut ke kolom penanda ( *flagging* ) tersendiri seperti `is_error = 1`. Gunakan angka persentil 99.9 hasil pembersihan (`2.045,91 MB`) sebagai ambang batas kontrol baru untuk *Early Warning System* deteksi anomali.
