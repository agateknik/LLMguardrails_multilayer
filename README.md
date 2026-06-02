# LLM Guardrails Multilayer

Repository ini berisi prototipe penelitian arsitektur **multilayered guardrails** untuk aplikasi chatbot berbasis Large Language Model (LLM). Fokus utama proyek ini adalah mitigasi risiko keamanan dan halusinasi pada sistem tanya jawab berbasis Retrieval-Augmented Generation (RAG), khususnya dalam konteks chatbot informasi seleksi pegawai.

Proyek ini merupakan bagian dari usulan arsitektur untuk penelitian tesis S2 Program Studi Teknik Elektro, Peminatan Manajemen Keamanan Jaringan Informasi, Universitas Indonesia.

## Latar Belakang

Aplikasi berbasis LLM memiliki potensi besar untuk membantu proses layanan informasi, tetapi juga membawa sejumlah risiko, seperti:

- prompt injection;
- jailbreak;
- permintaan data sensitif;
- kebocoran informasi pribadi;
- jawaban yang tidak relevan dengan konteks;
- halusinasi atau jawaban yang tidak didukung dokumen sumber.

Repository ini mengusulkan pendekatan **pertahanan berlapis** dengan menempatkan beberapa guardrail pada tahap input, konteks, dan output. Tujuannya adalah agar sistem tidak hanya menghasilkan jawaban yang relevan, tetapi juga lebih aman, terkendali, dan dapat dievaluasi secara sistematis.

## Tujuan Penelitian

Tujuan utama proyek ini adalah merancang dan menguji arsitektur guardrails berlapis untuk:

1. mendeteksi dan memblokir input berbahaya sebelum masuk ke pipeline LLM;
2. mengurangi risiko prompt injection dan jailbreak;
3. membatasi jawaban hanya berdasarkan dokumen yang tersedia pada sistem RAG;
4. mengevaluasi ketahanan sistem terhadap halusinasi;
5. melakukan masking terhadap informasi sensitif atau Personally Identifiable Information (PII);
6. menyediakan artefak eksperimen berupa dataset, model, notebook, dan hasil evaluasi.

## Gambaran Arsitektur

Secara umum, pipeline sistem terdiri dari beberapa lapisan berikut:

```text
User Input
   |
   v
[Small Talk Detection]
   |
   v
[Input Guardrail]
   |-- Prompt Guard
   |-- Regex Guard
   |-- ML Sensitive Guard
   |
   v
[RAG Pipeline]
   |-- Document Retrieval
   |-- Hybrid Retrieval
   |-- Cross-Encoder Re-ranking / Thresholding
   |
   v
[Output Guardrail]
   |-- Hallucination Judge
   |-- PII Masking
   |
   v
Final Answer
```

## Lapisan Guardrails

### 1. Small Talk Detection

Lapisan awal digunakan untuk mendeteksi percakapan ringan yang tidak perlu diproses oleh pipeline RAG. Jika input termasuk small talk, sistem dapat memberikan respons langsung tanpa melakukan retrieval dokumen.

### 2. Prompt Guard

Lapisan ini digunakan untuk mendeteksi potensi prompt injection atau jailbreak pada input pengguna. Dalam notebook prototipe, pendekatan ini menggunakan model Prompt Guard dari Meta Llama untuk mengklasifikasikan apakah input termasuk aman atau berisiko.

### 3. Regex Guard

Regex Guard digunakan sebagai lapisan deteksi berbasis pola. Guardrail ini berfungsi untuk mengenali frasa atau pola eksplisit yang mengarah pada instruksi berbahaya, permintaan manipulatif, atau upaya bypass sistem.

### 4. ML Sensitive Guard

Lapisan ini menggunakan model klasifikasi berbasis machine learning untuk mendeteksi input sensitif. Model disimpan dalam folder `model_sensitiveGuard/` dan menggunakan artefak seperti vectorizer TF-IDF serta model ensemble.

### 5. Context Guard / RAG Guard

Lapisan ini memastikan jawaban dihasilkan berdasarkan konteks dokumen yang tersedia. Jika konteks tidak ditemukan, sistem akan mengembalikan jawaban fallback agar tidak mengarang informasi.

### 6. Hallucination Judge

Output dari LLM diperiksa kembali menggunakan hallucination judge. Tujuannya adalah menilai apakah jawaban yang dihasilkan masih selaras dengan konteks dokumen yang digunakan sebagai referensi.

### 7. PII Masking

Lapisan akhir melakukan masking terhadap informasi sensitif atau data pribadi yang mungkin muncul pada jawaban. Lapisan ini penting untuk mengurangi risiko kebocoran informasi pribadi.

## Struktur Repository

```text
LLMguardrails_multilayer/
├── RAG_doc/
│   ├── ALOKASI_KEBUTUHAN_FORMASI.csv
│   ├── CEK_HASIL_KELULUSAN.md
│   ├── JADWAL_SELEKSI.csv
│   ├── KRITERIA_PELAMAR.md
│   ├── PENGUMUMAN.md
│   ├── PERSYARATAN_PENDAFTARAN.md
│   ├── TAHAPAN_SELEKSI.md
│   ├── TATA_CARA_PENDAFTARAN.md
│   ├── data_hasil_seleksi_dummy.csv
│   └── data_pribadi_peserta_dummy.csv
│
├── dataset_pelatihan_model_ensemble/
│   ├── safe_dataset.csv
│   └── sensitive_dataset.csv
│
├── dataset_pengujian_manual/
│   └── dataset_pengujian.xlsx
│
├── hasil _evaluasi/
│   ├── Wilson_Score_CI_HRR.xlsx
│   ├── Wilson_Score_CI_HallRR.xlsx
│   ├── uji_answer_relevancy.xlsx
│   ├── uji_garak_all.xlsx
│   ├── uji_keamanan.xlsx
│   └── uji_ketahanan_halusinasi.xlsx
│
├── model_sensitiveGuard/
│   ├── config.pkl
│   ├── ensemble_model.pkl
│   └── tfidf_vectorizer.pkl
│
├── src/
│   ├── [MASTER]_BaseChatBot+Guard(Input+Context+Output)_Prototype.ipynb
│   └── README.md
│
├── .gitignore
└── README.md
```

## Penjelasan Folder

### `RAG_doc/`

Berisi dokumen dan data yang digunakan sebagai knowledge base untuk sistem RAG. Folder ini mencakup informasi pengumuman, jadwal seleksi, kriteria pelamar, persyaratan pendaftaran, tahapan seleksi, tata cara pendaftaran, alokasi formasi, serta data dummy peserta dan hasil seleksi.

### `dataset_pelatihan_model_ensemble/`

Berisi dataset untuk melatih model klasifikasi sensitive guard. Dataset dipisahkan menjadi data aman (`safe_dataset.csv`) dan data sensitif (`sensitive_dataset.csv`).

### `dataset_pengujian_manual/`

Berisi dataset pengujian manual untuk mengevaluasi performa guardrails terhadap berbagai skenario pertanyaan atau serangan.

### `hasil _evaluasi/`

Berisi hasil evaluasi eksperimen, termasuk evaluasi keamanan, ketahanan halusinasi, answer relevancy, pengujian Garak, serta perhitungan Wilson Score Confidence Interval untuk metrik evaluasi.

### `model_sensitiveGuard/`

Berisi artefak model untuk ML Sensitive Guard, yaitu konfigurasi model, vectorizer TF-IDF, dan model ensemble yang digunakan untuk klasifikasi input sensitif.

### `src/`

Berisi notebook utama prototipe sistem. Notebook ini menggabungkan pipeline chatbot, RAG, input guardrails, context guardrails, output guardrails, serta pengujian fungsi sistem.

## Notebook Utama

Notebook utama berada pada:

```text
src/[MASTER]_BaseChatBot+Guard(Input+Context+Output)_Prototype.ipynb
```

Notebook ini berisi implementasi prototipe end-to-end, meliputi:

- setup model dan dependensi;
- pembuatan pipeline RAG;
- implementasi Prompt Guard;
- implementasi Regex Guard;
- implementasi ML Sensitive Guard;
- implementasi hallucination judge;
- implementasi PII masking;
- fungsi chatbot dengan guardrails berlapis;
- pengujian pertanyaan normal, pertanyaan berisiko, dan skenario evaluasi.

## Dataset dan Domain Eksperimen

Domain eksperimen pada repository ini menggunakan skenario chatbot informasi seleksi pegawai. Dokumen RAG dan data dummy disediakan untuk mensimulasikan sistem informasi yang dapat menjawab pertanyaan terkait:

- pengumuman seleksi;
- jadwal seleksi;
- tahapan seleksi;
- kriteria pelamar;
- persyaratan pendaftaran;
- tata cara pendaftaran;
- alokasi formasi;
- hasil seleksi peserta;
- data dummy peserta.

Penggunaan data dummy bertujuan untuk mendukung eksperimen tanpa menggunakan data pribadi nyata.

## Evaluasi

Evaluasi dilakukan untuk mengukur efektivitas arsitektur multilayered guardrails terhadap dua aspek utama:

1. **Keamanan**, yaitu kemampuan sistem dalam mendeteksi dan memblokir prompt injection, jailbreak, serta input sensitif.
2. **Ketahanan terhadap halusinasi**, yaitu kemampuan sistem untuk menolak atau mengoreksi jawaban yang tidak didukung oleh konteks dokumen.

File evaluasi pada folder `hasil _evaluasi/` mencakup hasil pengujian manual, pengujian dengan Garak, evaluasi answer relevancy, evaluasi hallucination resistance, dan perhitungan Wilson Score Confidence Interval.

## Contoh Alur Kerja Sistem

1. Pengguna mengirim pertanyaan.
2. Sistem memeriksa apakah input termasuk small talk.
3. Jika bukan small talk, input diperiksa oleh Prompt Guard.
4. Jika lolos, input diperiksa oleh Regex Guard.
5. Jika lolos, input diperiksa oleh ML Sensitive Guard.
6. Jika aman, sistem mengambil konteks dari dokumen RAG.
7. LLM menghasilkan jawaban berdasarkan konteks.
8. Hallucination Judge memeriksa kesesuaian jawaban dengan konteks.
9. PII Masking melakukan masking jika ditemukan informasi sensitif.
10. Sistem mengembalikan jawaban akhir kepada pengguna.

## Teknologi yang Digunakan

Repository ini menggunakan pendekatan berbasis Python dan Jupyter Notebook. Beberapa komponen yang digunakan dalam prototipe meliputi:

- Large Language Model untuk generasi jawaban;
- Retrieval-Augmented Generation (RAG);
- Prompt Guard untuk deteksi jailbreak atau prompt injection;
- regular expression untuk deteksi pola berisiko;
- machine learning classifier untuk sensitive guard;
- TF-IDF vectorizer;
- ensemble model;
- hallucination judge;
- PII masking;
- spreadsheet evaluasi untuk analisis eksperimen.

## Cara Menggunakan Repository

1. Clone repository:

```bash
git clone https://github.com/agateknik/LLMguardrails_multilayer.git
cd LLMguardrails_multilayer
```

2. Buka notebook utama:

```text
src/[MASTER]_BaseChatBot+Guard(Input+Context+Output)_Prototype.ipynb
```

3. Jalankan notebook secara berurutan untuk melakukan setup model, memuat dataset, membangun pipeline RAG, dan menjalankan pengujian guardrails.

4. Pastikan path ke folder berikut sudah sesuai di environment notebook:

```text
RAG_doc/
model_sensitiveGuard/
dataset_pelatihan_model_ensemble/
dataset_pengujian_manual/
hasil _evaluasi/
```

## Catatan Reproduksibilitas

Beberapa model eksternal dapat membutuhkan akses internet, autentikasi, atau environment komputasi tertentu. Jika notebook dijalankan di Google Colab atau local machine, pastikan dependensi, path file, dan ketersediaan model sudah disesuaikan.

## Batasan Proyek

Repository ini merupakan prototipe penelitian, sehingga belum ditujukan sebagai sistem produksi. Beberapa batasan yang perlu diperhatikan:

- domain pengujian masih terbatas pada skenario informasi seleksi pegawai;
- dataset menggunakan data dummy;
- efektivitas guardrails bergantung pada kualitas dataset, threshold, model, dan skenario serangan;
- evaluasi perlu diperluas dengan variasi prompt injection, jailbreak, dan skenario halusinasi yang lebih beragam.

## Konteks Akademik

Proyek ini dikembangkan sebagai bagian dari penelitian tesis S2 dengan fokus pada keamanan aplikasi berbasis LLM. Arsitektur multilayered guardrails yang diusulkan bertujuan untuk meningkatkan keamanan, keandalan, dan keterkendalian sistem chatbot berbasis RAG.

Institusi:

```text
Program Studi Teknik Elektro
Peminatan Manajemen Keamanan Jaringan Informasi
Universitas Indonesia
```

## Status Repository

Status repository saat ini adalah prototipe penelitian. Struktur folder, dataset, model, dan hasil evaluasi dapat terus diperbarui seiring pengembangan eksperimen tesis.

## Lisensi

Apache 2.0.

## Citation
<span style="font-variant: small-caps;">

If you use this work in your research, please cite:

Author: Pratama, Angga

Title: Development and Evaluation of Multi-layered Guardrails Architecture for Mitigating Security Risks and Hallucinations in Employee Recruitment Service Chatbots Based on Large Language Models (LLM) and Retrieval Augmented Generation (RAG)

Institution: Universitas Indonesia

Program: Master of Electrical Engineering – Information Network Security Management
</span>

Year: 2026
