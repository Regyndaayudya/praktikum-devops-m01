# Blameless Postmortem - Insiden Kegagalan Deployment Manual

## Ringkasan Insiden
Pada simulasi serah-terima artefak antara Dveloper dan Operations, terjadi beberapa kegagalan deployment:
1. Pustaka Flask tidak ditemukan (`ModuleNotFoundError`) pada pengujian manual JOB 2 karena ketiadaan berkas `requirements.txt`.
2. Berkas sumber utama (`src/app.py`) tertinggal di luar repositori lokal saat transisi ke JOB 4.
3. Berkas arsip kompresi lama sempat terkirim ulang ke sisi Operations.
4. Terjadi penolakan autentikasi saat melakukan `git push` ke GitHub karena penggunaan kata sandi biasa.

## Kronologi
1. Developer menyerahkan folder manual `serah-terima/` berisi `src/` dan `HANDOVER.md` tanpa `requirements.txt`.
2. Operations menguji dengan `python3 src/app.py`; eksekusi gagal akibat `ModuleNotFoundError: No module named 'flask'`.
3. Operations mencoba pasang paket mandiri namun terbentur isolasi sistem; sesi manual JOB 2 dinyatakan gagal.
4. Operations memverifikasi kode di lokal; pengujian di tab terminal kedua via `curl http://127.0.0.1:5000/health` berhasil merespons status `ok` (HTTP 200).
5. Peran ditukar (JOB 4); Developer menyusun `setup.sh`, namun berkas `src/app.py` tertinggal di luar repositori kerja.
6. Berkas `src/app.py` dipindahkan ke lokasi yang benar, lalu arsip dikirim ulang (sempat terkirim arsip `.tar.gz` usang sebelum diperbarui).
7. Operations mengeksekusi `./setup.sh`; skrip otomatis membuat virtual environment, memasang dependensi, dan uji `curl` ke `/health` berhasil memunculkan status `ok`.

## Dampak
1. Waktu Terbuang: Proses deployment sekitar 15 menit
2. Jumlah kegagalan: Terjadi 4 kali insiden kegagalan prosedur (missing library, file tertinggal, arsip usang, dan autentikasi Git)

## Akar Masalah pada SISTEM (bukan pada orang)
1. Serah-terima manual (JOB 2) tidak menyertakan file manifes `requirements.txt`.
2. Pemisahan Peran Tanpa Standarisasi Lingkungan (Wall of Confusion):Asumsi aplikasi berjalan di mesin pengembang tanpa ada skrip otomasi yang menjamin replikasi di sisi operasional.
3. Distribusi kode awal tidak langsung memanfaatkan remote Git. 

## Tindakan Perbaikan (action items) + penanggung jawab peran
1. Otomasi Penuh via `setup.sh` (Peran: Developer & Operations):Mengganti prosedur manual dengan skrip yang otomatis memasang virtual environment, menginstal dependensi, dan menjalankan smoke test.
2. Uji Otomatis Endpoint `/health` (Peran: Operations):Memvalidasi respons HTTP 200 secara terprogram di dalam skrip penyebaran.
3. Sentralisasi Repositori via Git (Peran: Developer & Operations):Menghentikan transfer arsip manual dan beralih penuh ke repositori Git terpusat dengan konfigurasi `.gitignore`.
4. Standardisasi Akses CLI (Peran: Operations):Menetapkan penggunaan Personal Access Token (PAT) untuk autentikasi terminal ke remote repository GitHub.

## Pelajaran yang Diambil
1. Kegagalan deployment terjadi pada proses manual dan ketiadaan standarisasi lingkungan, bukan kelalaian individu.
2. Penerapan prinsip *The First Way: Flow* melalui otomasi terbukti dapat memotong Lead Time dan meniadakan kegagalan rilis.
 
