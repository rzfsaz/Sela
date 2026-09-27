# Sela

**Prarilis untuk Apple Silicon • macOS 14+**

[Unduh aplikasi Apple Silicon](https://github.com/rzfsaz/Sela/releases) atau [unduh proyek SwiftUI lengkap](Sela-Source.zip). Ekstrak ZIP proyek terlebih dahulu sebelum membuka Sela.xcodeproj. Kode sumber tersedia untuk dibangun sendiri. Paket aplikasi prarilis menggunakan ad-hoc signing dan belum dinotarisasi Apple, sehingga Gatekeeper dapat memblokir pemasangan. Untuk distribusi umum tanpa hambatan tersebut diperlukan Developer ID dan notarization.

Konfirmasi Power Timer tampil langsung di panel: pilih aksi dan durasi → Jadwalkan → Mulai timer. Menutup panel tidak membatalkan countdown; keluar dari aplikasi membatalkannya.

Utilitas pribadi yang tinggal di menu bar. SwiftUI native, Apple Silicon (`arm64`, termasuk M3), minimum macOS 14. Tidak ada WindowGroup atau ikon Dock. Membuka aplikasi dari Finder/Applications menampilkan panel sementara di kanan atas; ikon MenuBarExtra membuka panel persisten yang sama. Satu instance dijaga dengan file lock agar membuka salinan aplikasi mengarahkan ke panel yang sudah berjalan. Panel 600 × 504 pt menampung semua fitur; MenuBarExtra menjadi pintu masuk dari menu bar.

## Jalankan

- Buka `Sela.xcodeproj` menggunakan Xcode 15 atau lebih baru.
- Pilih scheme **Sela**, tujuan **My Mac**, lalu Run. Klik ikon lingkaran setengah terisi di menu bar.
- Atau jalankan `./Scripts/build.sh`, kemudian buka `build.noindex/Build/Products/Release/Sela.app`.
- Pasang hanya satu salinan aplikasi di folder Applications. Tutup versi lama sebelum menggantinya.

Build lokal memakai ad-hoc signing dan tidak memerlukan Apple Developer Team. Untuk distribusi ke Mac lain, gunakan identitas Developer ID Anda, hardened runtime, dan notarization. Build ini **belum dinotarisasi**. Jangan menjalankan binary Swift Package secara langsung untuk penggunaan normal: bundle `.app` dibutuhkan untuk identitas izin dan notifikasi macOS.

```sh
./Scripts/build.sh
./Scripts/test.sh
```

Tidak ada dependensi pihak ketiga. Swift Package dipakai untuk kompilasi tambahan dan XCTest. Proyek Xcode mengompilasi sumber yang sama sebagai satu modul aplikasi. `#if SWIFT_PACKAGE` memisahkan import `SelaCore` pada build paket.

## Fitur dan perilaku

| Modul | Implementasi |
| --- | --- |
| Greeting | `NSFullUserName()`, fallback `NSUserName()`, sapaan pagi/siang/sore/malam, 1,5 detik atau klik untuk lewati. Lifecycle mengamati jendela MenuBarExtra karena SwiftUI dapat mempertahankan view antar pembukaan. Reduce Motion dihormati. |
| Power Timer | Sleep, Shutdown, Restart, Log Out; preset 15/30/60/90 menit dan picker jam/menit. Countdown di menu bar. Konfirmasi sebelum penjadwalan. Cancel dan tunda 10 menit. Peringatan 5 dan 1 menit dengan aksi notifikasi. |
| Keep Awake | IOKit assertion `PreventUserIdleSystemSleep`, selamanya/30 menit/1 jam/sampai proses aplikasi pilihan berhenti. Layar masih dapat redup. Assertion dilepas saat dimatikan, keluar, atau sebelum aksi daya. |
| Battery Health | IOPowerSources untuk charge, kondisi dan estimasi waktu; registry AppleSmartBattery untuk cycle count dan rasio kapasitas. Pengingat opsional setelah sekitar 2 jam tersambung di 100%. |
| Clipboard | Opt-in, hingga 15 teks/gambar, deduplikasi SHA-256. Klik untuk menyalin kembali; paste otomatis opsional dengan Accessibility. Expiry 5/30/60 menit atau sampai keluar. |
| Quick Cleaner | Preview cache/log pengguna yang tidak berubah ≥7 hari serta Trash opsional. Pilih kategori, tinjau nama file dan ukuran, lalu konfirmasi penghapusan permanen. |
| Network | Pembaruan per detik melalui `getifaddrs`, grafik 40 sampel, upload/download opsional di menu bar. |
| Quick Notes | Editor langsung di popover; perubahan disimpan otomatis di UserDefaults. |
| Focus Timer | Fokus 15/25/45/50 menit, jeda, lanjut, selesai, istirahat 5 menit. Deadline dan DispatchSourceTimer terpisah dari Power Timer. Integrasi DND melalui Shortcuts. |

## Izin dan setup pertama

### Notifikasi

Timer langsung dimulai setelah konfirmasi dan tidak menunggu izin notifikasi. Izin peringatan dapat diminta secara terpisah melalui tombol Aktifkan peringatan atau Pengaturan → Izinkan notifikasi. Izin yang ditolak atau belum dijawab tidak menahan countdown; status ditampilkan di UI. Setelah penolakan, ubah di System Settings → Notifications → Sela. Banner dan suara tetap mengikuti pengaturan Focus macOS. Timer pendek tidak menjadwalkan peringatan yang waktunya sudah lewat.

### Power Timer

Aksi menggunakan `/usr/bin/osascript` dengan skrip tetap untuk System Events, tanpa shell interpolation, `sudo`, atau force-kill. macOS dapat meminta izin Automation saat pertama kali aksi dijalankan. Bagian Akses Sleep / Shutdown menyediakan tombol Periksa akses macOS. Pemeriksaan menjalankan permintaan versi System Events yang tidak melakukan aksi daya, memakai jalur proses yang sama dengan aksi sebenarnya. Penolakan Automation ditampilkan dengan petunjuk pengaturan. Build saja tidak membuktikan aksi daya akan berhasil.

Shutdown/Restart/Log Out dapat dihentikan oleh aplikasi dengan dokumen belum disimpan. Sela melaporkan bahwa permintaan dikirim, bukan mengklaim sistem sudah dimatikan. Tidak ada retry otomatis. Pesan error Automation muncul di popover dan dikirim sebagai notifikasi bila diizinkan.

Timer memerlukan Sela berjalan dan Mac terjaga. Sela tidak menjadwalkan wake-up sistem. Aktifkan Keep Awake bila perlu. Countdown menggunakan waktu akhir absolut, tidak mengurangi angka secara buta per tick. Timer yang terlewat ketika Mac tidur dibatalkan setelah wake. Timer sesi sebelumnya juga dibatalkan saat aplikasi diluncurkan kembali untuk menghindari aksi daya mendadak. Perubahan jam sistem memengaruhi deadline.

### Clipboard

Aktifkan **Simpan riwayat sesi**. Tidak ada pembacaan histori sebelum aktivasi; hanya perubahan clipboard berikutnya yang direkam. Menonaktifkan fitur menghapus riwayat memori. **Hapus semua** menghapus riwayat Sela, bukan clipboard sistem.

**Paste otomatis saat diklik** memerlukan System Settings → Privacy & Security → Accessibility → Sela. Tanpa izin, klik tetap menyalin item sehingga Anda dapat menggunakan ⌘V sendiri. Dengan izin, Sela kembali ke aplikasi yang terakhir aktif dan hanya mengirim ⌘V bila aplikasi tersebut benar-benar kembali menjadi foreground. Clipboard tidak dikirim ke server dan tidak disimpan di disk. Penanda clipboard concealed/transient/autogenerated dilewati; penanda ini tidak menjamin semua rahasia terdeteksi. Batas per item: teks 100 KB, gambar 8 MB dan 40 megapiksel. Memori tidak dapat dijanjikan bebas swap sistem.

### Do Not Disturb / Focus

Integrasi tersedia tetapi memerlukan **dua shortcut buatan pengguna**. Aplikasi tidak menulis database Focus atau memakai API privat. Buat di aplikasi Shortcuts:

1. **Sela · Focus On** — tambahkan **Set Focus → Turn Do Not Disturb On → Until Turned Off**.
2. **Sela · Focus Off** — tambahkan **Set Focus → Turn Do Not Disturb Off**.
3. Cocokkan nama di Focus Timer → Pengaturan Shortcuts, lalu aktifkan **Do Not Disturb otomatis**.

Sela memeriksa ketersediaan kedua nama, menjalankan `shortcuts run` untuk On, kemudian memulai timer. Shortcut Off berjalan saat jeda, selesai, cancel, atau keluar normal. Shortcut sebaiknya tidak mengandung prompt interaktif; proses dibatasi sekitar 45 detik. Kegagalan ditampilkan dan tidak dianggap berhasil. Jika sesi terputus oleh crash atau force quit, tombol **Pulihkan Focus** tersedia pada peluncuran berikutnya.

Mode default Off mematikan DND; tidak memulihkan Focus yang sebelumnya aktif. Pengguna dapat mengganti shortcut Off dengan alur pemulihan sendiri. Timer fokus dapat dipakai tanpa integrasi DND. Sistem macOS tidak menyediakan API SwiftUI umum untuk mengaktifkan global DND; Focus Filter adalah integrasi terhadap mode yang dipilih pengguna, bukan penggantinya.

### Cleaner

Scope dibatasi ke `~/Library/Caches`, `~/Library/Logs`, dan `~/.Trash`. Trash tidak dipilih secara default. Tidak ada eskalasi izin otomatis. Folder yang ditolak macOS dilaporkan dan dilewati. Beberapa data Trash mungkin membutuhkan Full Disk Access bila Anda ingin mengizinkannya melalui Settings; Cleaner tetap dapat digunakan untuk file yang tersedia.

Penghapusan hanya berlaku untuk regular file dalam snapshot preview. Symlink, hardlink, file yang berubah, dan file yang identitasnya berbeda dilewati. Traversal menggunakan descriptor `openat` dengan `O_NOFOLLOW`; inode, device, ukuran, mtime, dan ctime diperiksa ulang sebelum `unlinkat`. Ini mengurangi risiko balapan filesystem, namun bukan jaminan transaksi atomik terhadap proses lain yang sedang mengubah file. Tutup aplikasi terkait sebelum membersihkan.

File dihapus permanen setelah konfirmasi. Direktori kosong dan symlink di Trash dibiarkan; fitur ini tidak menjanjikan Trash sepenuhnya kosong. Preview dibatasi 20.000 file. Total hasil adalah ukuran alokasi file yang berhasil dihapus, bukan janji perubahan ruang disk identik: APFS, clone, kompresi, dan snapshot memengaruhi ruang aktual. Tidak ada file pribadi yang dihapus saat pengembangan/pengujian proyek ini.

### Battery & Network

Field registry baterai bukan kontrak stabil di semua perangkat. Data yang tidak tersedia ditampilkan sebagai “Tidak tersedia”; health % adalah estimasi, bukan jaminan sama dengan System Settings. Desktop Mac tanpa baterai menampilkan empty state. Pengingat 100% bukan kontrol pengisian atau diagnosis baterai; fitur tidak menonaktifkan Optimized Battery Charging.

Network menjumlahkan antarmuka fisik `en*` yang aktif. Loopback, bridge, dan tunnel VPN tidak dijumlahkan lagi agar tidak menghitung trafik yang sama dua kali. Bukan pemantau per aplikasi dan bukan pengukur kapasitas koneksi. Reset counter dan pergantian interface menghasilkan baseline baru, bukan lonjakan palsu.

## Struktur proyek

```text
Sela.xcodeproj/                 Project & shared scheme untuk aplikasi
Package.swift                  Build Swift Package & unit test
Resources/
  Info.plist                   LSUIElement, minimum macOS, pesan Automation
  Sela.entitlements            Apple Events; build di luar App Sandbox
Sources/
  SelaCore/
    TimerPlan.swift            Deadline, aksi daya, format, perhitungan jaringan
    SafeCleaner.swift          Preview & penghapusan dengan validasi identitas
  Sela/
    App/
      SelaApp.swift            Entry point MenuBarExtra dan label countdown
      AppModel.swift          Komposisi layanan & cleanup saat keluar
      LaunchPanelController.swift  Panel saat launch/reopen dan pencegahan instance ganda
      PopoverRoot.swift       Sidebar 8 fitur, preferences, transisi
      PopoverLifecycle.swift  Deteksi buka/tutup panel native
      GreetingView.swift      Sapaan personal
      PreviewExport.swift     Render offscreen khusus build Debug
    Shared/
      Infrastructure.swift    DispatchTicker, proses, notifikasi, histori
      Components.swift        Komponen visual native bersama
    Features/
      PowerTimer/             Service & view
      KeepAwake/              Service & view
      Battery/                Service & view
      Clipboard/              Service & view
      Cleaner/                Service & view
      Network/                Service & view
      Notes/                  Service & view
      Focus/                  Service & view
Tests/SelaCoreTests/           28 XCTest, termasuk regresi service timer
Scripts/                      Build & test
```

State layanan bertahan ketika popover ditutup. View tiap fitur hanya mengamati layanannya. Pemrosesan command dan scan/hapus filesystem dilakukan di luar main thread. Semua repeating timer memakai DispatchSourceTimer; delay sapaan dan transisi paste menggunakan Task sleep yang tidak memblokir UI. Preferensi, catatan, pending state untuk recovery, dan maksimal 40 event aktivitas menggunakan UserDefaults. Tidak ada telemetry, server, atau SDK analitik.

Untuk menambah fitur: buat service dan view dalam subfolder `Features`, daftarkan service di `AppModel`, tambahkan section di `PopoverRoot`, dan tambahkan file baru ke target Xcode. Swift Package menemukan sumber di dalam direktori target otomatis.

## Verifikasi

Build Release arm64 dan Debug berhasil pada toolchain Xcode di Mac pengembangan. `swift test` menjalankan 28 pengujian: deadline dan postpone, waktu peringatan, persistence, format/sapaan, counter jaringan, pemilihan cache tua, hardlink/symlink, file berubah, penggantian inode/ancestor, batas preview, dan penghapusan file fixture.

Pengujian UI langsung sudah mencakup buka dari Finder, tutup panel, buka ulang, penjadwalan Sleep dan Shutdown, tunda 10 menit, cancel, serta pemeriksaan akses System Events. Countdown dimulai tanpa menunggu izin notifikasi. Semua timer uji dibatalkan sebelum berakhir. Belum diuji langsung di macOS 14 atau perangkat M3 tertentu. Aksi shutdown/sleep/restart/log out, penghapusan file pengguna, paste ke aplikasi lain, dan perubahan DND nyata sengaja tidak dijalankan saat validasi.

Checklist uji manual setelah izin tersedia:

- Buka/tutup popover berulang; greeting muncul lagi dan dapat dilewati.
- Periksa Light/Dark, Reduce Motion, keyboard, dan teks tiap tab.
- Jadwalkan 15 menit lalu cancel dan tunda; pastikan countdown dan notifikasi lama hilang.
- Aktifkan Keep Awake sebentar; pastikan berhenti saat dimatikan atau app target keluar.
- Uji Clipboard dengan teks/gambar nonrahasia, lalu periksa auto-clear dan paste.
- Tulis catatan, tutup/buka Sela, pastikan isi tetap tersimpan.
- Siapkan dua Shortcuts, lalu mulai/jeda/cancel Focus Timer.
- Tinjau Cleaner dengan preview terlebih dahulu; uji penghapusan hanya pada data yang memang boleh dihapus.

## Referensi Apple

- [MenuBarExtra](https://developer.apple.com/documentation/swiftui/menubarextra)
- [Window menu bar style](https://developer.apple.com/documentation/swiftui/menubarextrastyle/window)
- [IOPMAssertionTypes](https://developer.apple.com/documentation/iokit/iopmlib_h/iopmassertiontypes)
- [IOPS Battery Health key](https://developer.apple.com/documentation/iokit/kiopsbatteryhealthkey)
- [Accessibility trust](https://developer.apple.com/documentation/applicationservices/1459186-axisprocesstrustedwithoptions)
- [Menjalankan Shortcuts dari command line](https://support.apple.com/en-gb/guide/shortcuts-mac/apd455c82f02/mac)
- [Focus intents](https://developer.apple.com/documentation/appintents/focus)

## Pemakaian sehari-hari

Gunakan hanya **Applications → Sela** atau tulisan **Sela** di menu bar. Tombol **×** di kiri bawah menutup panel; **Esc** juga menutupnya. Tombol power **Keluar dari Sela** menghentikan aplikasi. Panel dapat dibuka ulang dari Applications meskipun Sela sudah berjalan.

Perbaikan timer memisahkan izin notifikasi dari penjadwalan. Regression test memastikan start langsung menghasilkan deadline, expiry memanggil executor sekali, cancel mencegah eksekusi, dan kegagalan terlihat tanpa retry otomatis. Unit test menggunakan executor palsu; tidak mematikan atau menidurkan komputer.
