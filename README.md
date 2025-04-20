# Guide: Fixing `libmysqlclient.so.18` Error for SA-MP Server on Newer Ubuntu Versions

## Introduction (English)

This is a guide to resolve the common error `libmysqlclient.so.18: cannot open shared object file: No such file or directory`. This issue often occurs when running a SA-MP (San Andreas Multiplayer) server (`samp03svr`) with an older MySQL plugin (like R41-4) on newer Ubuntu versions (e.g., 20.04 LTS, 22.04 LTS, 24.04 LTS, and later).

The error happens due to an incompatibility between the library dependencies required by the old MySQL plugin and the libraries available on modern Ubuntu systems.

## Pendahuluan (Bahasa Indonesia)

Ini adalah panduan untuk mengatasi error umum `libmysqlclient.so.18: cannot open shared object file: No such file or directory` yang sering muncul ketika menjalankan server SA-MP (`samp03svr`) dengan plugin MySQL (khususnya versi lama seperti R41-4) pada sistem operasi Ubuntu versi baru (misalnya 20.04 LTS, 22.04 LTS, 24.04 LTS, dan seterusnya).

Error ini terjadi karena ketidakcocokan antara dependensi library yang dibutuhkan oleh plugin MySQL lama dengan library yang tersedia di sistem Ubuntu modern.

## Cause of the Error (English)

This problem typically arises from a combination of these factors:

* **Old Plugin:** The `mysql.so` plugin you are using (e.g., R41-4) was compiled against a specific MySQL client library version, `libmysqlclient.so.18`, which originates from the MySQL 5.6 era.
* **New Ubuntu:** Standard repositories in newer Ubuntu versions no longer include the `libmysqlclient18` package. They use newer library versions, such as `libmysqlclient.so.21` (from MySQL 8.0) or `libmariadb.so.3` (from MariaDB).
* **32-bit Architecture:** The SA-MP server, many of its plugins (including older `mysql.so`), and the required `libmysqlclient.so.18` are typically **32-bit (`i386`)** applications. If your Ubuntu system is 64-bit (`amd64`), you not only need the correct library but also its 32-bit version and other base 32-bit libraries (`libc6:i386`, `libstdc++6:i386`, etc.).

## Penyebab Error (Bahasa Indonesia)

Masalah ini biasanya disebabkan oleh kombinasi faktor berikut:

* **Plugin Lama:** Plugin `mysql.so` yang Anda gunakan (misalnya R41-4) dikompilasi untuk menggunakan library klien MySQL versi spesifik, yaitu `libmysqlclient.so.18`, yang berasal dari era MySQL 5.6.
* **Ubuntu Baru:** Repositori standar Ubuntu versi baru tidak lagi menyertakan paket `libmysqlclient18`. Mereka menggunakan versi library yang lebih baru, seperti `libmysqlclient.so.21` (dari MySQL 8.0) atau `libmariadb.so.3` (dari MariaDB).
* **Arsitektur 32-bit:** Server SA-MP dan banyak pluginnya (termasuk `mysql.so` versi lama dan `libmysqlclient.so.18` yang dibutuhkannya) adalah aplikasi **32-bit (`i386`)**. Jika sistem Ubuntu Anda 64-bit (`amd64`), Anda tidak hanya membutuhkan library yang tepat, tetapi juga versi 32-bit-nya dan library dasar 32-bit lainnya (`libc6:i386`, `libstdc++6:i386`, dll.).

## Solution: Manual Library Installation (English)

The most common and effective solution is to **manually** install the 32-bit (`i386`) version of the `libmysqlclient.so.18` library by downloading the package from an older Ubuntu release.

## Solusi: Instalasi Manual Library (Bahasa Indonesia)

Solusi yang paling umum dan efektif adalah menginstal library `libmysqlclient.so.18` versi 32-bit (`i386`) secara **manual** dengan mengunduh paket dari rilis Ubuntu yang lebih lama.

## Manual Installation Steps (English)

Here are the detailed steps to perform the manual installation:

1.  **Ensure i386 Architecture is Enabled:**
    If not already done, enable support for 32-bit packages:
    ```bash
    # Enable 32-bit architecture
    sudo dpkg --add-architecture i386
    # Update package list
    sudo apt update
    # Install base 32-bit libraries (essential)
    sudo apt install libc6:i386 libstdc++6:i386 -y 
    ```

2.  **Download the `.deb` Package:**
    Obtain the `.deb` file for `libmysqlclient18` (i386 architecture) from an older Ubuntu release repository. A verified source is Ubuntu 12.04 (Precise). You can download it from Launchpad:
    * **Visit Page:** [Ubuntu 12.04 (Precise) - libmysqlclient18 (i386)](https://launchpad.net/ubuntu/precise/i386/libmysqlclient18/5.5.24-0ubuntu0.12.04.1)
    * **Download File:** Find and download the `.deb` file from that page. The filename will be similar to `libmysqlclient18_5.5.24-0ubuntu0.12.04.1_i386.deb`. You can use `wget` in the terminal if you have the direct download URL.

3.  **Extract the `.deb` File:**
    **Important:** Do not install this `.deb` package directly using `dpkg -i`, as it might cause dependency conflicts on your newer system. We only need to extract the library file.
    ```bash
    # Navigate to the directory where you downloaded the .deb file (e.g., ~/Downloads)
    cd ~/Downloads 

    # Create a temporary directory for extraction
    mkdir temp_lib_mysql

    # Extract the contents of the .deb package into the temporary directory
    # Use the full filename or a wildcard * if it's the only .deb file there
    dpkg -x libmysqlclient18_*.deb temp_lib_mysql/ 
    ```

4.  **Copy the Library File:**
    Copy the actual library `.so` file (usually the one with the full version number) from the extracted directory to your system's 32-bit library directory (`/usr/lib/i386-linux-gnu/`).
    ```bash
    # Check the exact filename inside temp_lib_mysql/usr/lib/i386-linux-gnu/
    # It's typically libmysqlclient.so.18.0.0
    ls temp_lib_mysql/usr/lib/i386-linux-gnu/ 

    # Copy the library file to the system location (ensure the target directory exists)
    sudo cp temp_lib_mysql/usr/lib/i386-linux-gnu/libmysqlclient.so.18.0.0 /usr/lib/i386-linux-gnu/
    ```

5.  **Create the Symbolic Link:**
    The MySQL plugin usually looks for the library name without the full version number (`libmysqlclient.so.18`). Create a symbolic link (symlink) with that name pointing to the actual library file you just copied. The `ln -sf` command will overwrite any existing link (like an incorrect one pointing to `libmariadb.so.3`).
    ```bash
    # -s creates a symbolic link
    # -f (force) overwrites if the link name already exists
    sudo ln -sf /usr/lib/i386-linux-gnu/libmysqlclient.so.18.0.0 /usr/lib/i386-linux-gnu/libmysqlclient.so.18
    ```

6.  **Update the Linker Cache:**
    Inform the system about the new library's location so applications can find it.
    ```bash
    sudo ldconfig
    ```

7.  **Clean Up (Optional):**
    You can remove the downloaded `.deb` file and the temporary directory if they are no longer needed.
    ```bash
    # Navigate back to the Downloads directory (or wherever you downloaded the file)
    cd ~/Downloads 

    # Remove the .deb file and the temporary directory
    rm libmysqlclient18_*.deb
    rm -rf temp_lib_mysql
    ```
    
## Langkah-langkah Instalasi Manual (Bahasa Indonesia)

Berikut adalah langkah-langkah rinci untuk melakukannya:

1.  **Pastikan Arsitektur i386 Aktif:**
    Jika belum, aktifkan dukungan untuk paket 32-bit:
    ```bash
    # Aktifkan arsitektur 32-bit
    sudo dpkg --add-architecture i386
    # Update daftar paket
    sudo apt update
    # Install library dasar 32-bit (penting)
    sudo apt install libc6:i386 libstdc++6:i386 -y 
    ```

2.  **Unduh Paket `.deb`:**
    Dapatkan file `.deb` untuk `libmysqlclient18` arsitektur `i386` dari rilis Ubuntu yang lebih lama. Sumber yang terverifikasi adalah Ubuntu 12.04 (Precise). Anda bisa mengunduhnya dari Launchpad:
    * **Kunjungi Halaman:** [Ubuntu 12.04 (Precise) - libmysqlclient18 (i386)](https://launchpad.net/ubuntu/precise/i386/libmysqlclient18/5.5.24-0ubuntu0.12.04.1)
    * **Unduh File:** Cari dan unduh file `.deb` dari halaman tersebut. Nama filenya akan mirip seperti `libmysqlclient18_5.5.24-0ubuntu0.12.04.1_i386.deb`. Anda bisa menggunakan `wget` di terminal jika sudah mendapatkan URL download langsung.

3.  **Ekstrak File `.deb`:**
    **Penting:** Jangan instal paket `.deb` ini secara langsung menggunakan `dpkg -i` karena dapat menyebabkan konflik dependensi di sistem baru Anda. Kita hanya perlu mengekstrak file library-nya.
    ```bash
    # Pindah ke direktori tempat Anda mengunduh file .deb (misal: ~/Downloads)
    cd ~/Downloads 

    # Buat direktori sementara untuk ekstraksi
    mkdir temp_lib_mysql

    # Ekstrak isi paket .deb ke direktori sementara
    # Gunakan nama file lengkap atau wildcard * jika hanya ada satu file .deb di sana
    dpkg -x libmysqlclient18_*.deb temp_lib_mysql/ 
    ```

4.  **Salin File Library:**
    Salin file library `.so` yang sebenarnya (biasanya dengan nomor versi lengkap) dari direktori hasil ekstraksi ke direktori library 32-bit sistem Anda (`/usr/lib/i386-linux-gnu/`).
    ```bash
    # Cek nama file persisnya (umumnya libmysqlclient.so.18.0.0)
    ls temp_lib_mysql/usr/lib/i386-linux-gnu/ 

    # Salin file library ke lokasi sistem (pastikan direktori tujuan ada)
    sudo cp temp_lib_mysql/usr/lib/i386-linux-gnu/libmysqlclient.so.18.0.0 /usr/lib/i386-linux-gnu/
    ```

5.  **Buat Symbolic Link:**
    Plugin MySQL biasanya mencari nama library tanpa nomor versi lengkap (`libmysqlclient.so.18`). Buat symbolic link (symlink) dengan nama tersebut yang menunjuk ke file library yang baru saja Anda salin. Perintah `ln -sf` akan menimpa link lama jika ada (misalnya link ke `libmariadb.so.3` yang salah).
    ```bash
    # -s membuat symbolic link
    # -f (force) berguna untuk menimpa link lama jika sudah ada
    sudo ln -sf /usr/lib/i386-linux-gnu/libmysqlclient.so.18.0.0 /usr/lib/i386-linux-gnu/libmysqlclient.so.18
    ```

6.  **Update Cache Linker:**
    Beri tahu sistem tentang lokasi library baru agar dapat ditemukan oleh aplikasi.
    ```bash
    sudo ldconfig
    ```

7.  **Bersihkan (Opsional):**
    Anda bisa menghapus file `.deb` yang diunduh dan direktori sementara jika sudah tidak diperlukan.
    ```bash
    # Kembali ke direktori Downloads (atau tempat Anda mengunduh)
    cd ~/Downloads 

    # Hapus file .deb dan direktori sementara
    rm libmysqlclient18_*.deb
    rm -rf temp_lib_mysql
    ```

## Verification (Important) / Verifikasi (Penting)

**(English)** After completing the steps above, verify that the system can now find the required library for your `mysql.so` plugin:
```bash
# Replace /path/to/your/samp/server/ with the actual path to your SA-MP server directory
ldd /path/to/your/samp/server/plugins/mysql.so 
```
Check the output of the `ldd` command. Look for the line mentioning `libmysqlclient.so.18`. It should no longer show `=> not found`, but instead point to the correct file path (`/usr/lib/i386-linux-gnu/libmysqlclient.so.18`).

**(Bahasa Indonesia)** Setelah semua langkah di atas selesai, verifikasi apakah sistem sekarang dapat menemukan library yang dibutuhkan oleh plugin `mysql.so` Anda:
```bash
# Ganti /path/to/your/samp/server/ dengan path sebenarnya ke direktori server SA-MP Anda
ldd /path/to/your/samp/server/plugins/mysql.so 
```
Periksa output dari perintah `ldd`. Cari baris yang menyebutkan `libmysqlclient.so.18`. Seharusnya sekarang tidak lagi menampilkan `=> not found`, melainkan menunjuk ke path file yang benar (`/usr/lib/i386-linux-gnu/libmysqlclient.so.18`).

## Conclusion (English)

After following these steps, try running your SA-MP server (`./samp03svr`) again. The `libmysqlclient.so.18` error should now be resolved, and the MySQL plugin should load correctly.

Hope this guide helps!

## Penutup (Bahasa Indonesia)

Setelah mengikuti langkah-langkah ini, coba jalankan kembali server SA-MP Anda (`./samp03svr`). Error `libmysqlclient.so.18` seharusnya sudah teratasi dan plugin MySQL dapat dimuat dengan benar.

Semoga panduan ini membantu!
