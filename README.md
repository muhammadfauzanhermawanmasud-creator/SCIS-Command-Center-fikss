# SCIS Control Tower — v24.1

Dashboard operasional untuk **Smart Circular Intake System**: intake PLTM jenis *run-of-river* dengan boom adaptif, trash rack, collection cage, conveyor, lini dewatering–sorting, dan jalur pemulihan material.

Dibangun ulang dari spesifikasi V23. Satu berkas HTML, tanpa *build step*, tanpa *framework*, tanpa dependensi yang perlu dipasang.

---

## Menjalankan di VS Code

1. Ekstrak ZIP.
2. **File → Open Folder** → pilih folder `scis-control-tower-v24`.
3. Klik kanan `index.html` → **Open with Live Server**.

VS Code akan menyarankan ekstensi Live Server secara otomatis (lihat `.vscode/extensions.json`). Port sudah diatur ke `5510` di `.vscode/settings.json`.

Tanpa Live Server pun dashboard tetap jalan: klik dua kali `index.html` dan browser akan membukanya langsung dari *file system*.

## Menaruh di GitHub Pages

Unggah **isi folder** (bukan foldernya) ke root repository:

```
index.html
manifest.webmanifest
.nojekyll
assets/icon.svg
```

Lalu **Settings → Pages → Deploy from branch → main / (root)**. Berkas `.nojekyll` sudah disertakan supaya Pages tidak memproses ulang isi folder.

### Perbaikan di v24.1

Bingkai grafik dikunci pada ukuran tetap. Sebelumnya tinggi kanvas dibaca ulang dari atribut `height`, padahal menyetel `canvas.height` menulis balik ke atribut yang sama — sehingga pada layar dengan *device pixel ratio* di atas 1 (Retina, layar HiDPI, atau browser yang di-*zoom*) bingkainya dikalikan ulang setiap kali digambar dan halaman memanjang tanpa henti. Tinggi yang dituju kini disimpan di atribut `data-h` yang tidak pernah ditulis ulang, lebar diambil dari kontainer induk, dan seluruh item grid diberi `min-width:0` agar lebar kanvas tidak bisa mendorong lebar kolomnya sendiri.

Sudah diuji stabil selama 60 detik pada pixel ratio 1, 1,5, 2, dan 3, termasuk setelah jendela diperkecil di tengah jalan.

---

## Struktur berkas

```
scis-control-tower-v24/
├── index.html              seluruh dashboard: markup, CSS, engine, render
├── manifest.webmanifest    metadata PWA (bisa dipasang sebagai aplikasi)
├── .nojekyll               agar GitHub Pages menyajikan berkas apa adanya
├── assets/
│   └── icon.svg            ikon aplikasi
└── .vscode/
    ├── settings.json       konfigurasi Live Server
    └── extensions.json     rekomendasi ekstensi
```

---

## Yang berubah dari V23

**Satu rantai data, bukan enam halaman terpisah.** Setiap angka di layar diturunkan dari satu model proses yang sama. Curah hujan menaikkan debit; debit menaikkan beban sampah; boom menangkap sebagian dan sisanya menumpuk di rack; tumpukan menaikkan ΔP; ΔP menurunkan *head* dan karena itu menurunkan produksi; conveyor menguras cage; hasil sorting menentukan recovery rate, jejak karbon, dan akhirnya angka rupiah di halaman *business case*. Tidak ada grafik hiasan yang berdiri sendiri.

**Process chain menempel di atas setiap halaman.** Tujuh simpul — inflow, boom, rack, cage, conveyor, sorting, recovery — selalu terlihat, lengkap dengan penanda simpul mana yang sedang menjadi *constraint*. Operator tidak perlu berpindah halaman untuk tahu di mana tekanannya.

**Decision engine dipisahkan dari tampilan.** Delapan aturan dievaluasi tiap *tick* dan menghasilkan daftar tindakan, bukan sekadar alarm. Dalam mode AUTO engine mengeksekusi; dalam MODE MANUAL engine hanya menyarankan dan menunggu perintah operator. Setiap perintah tercatat di *interlock & sequence log* beserta alasannya.

**Model yang dapat diaudit.** Ambang batas, faktor emisi, tarif, dan kapasitas dikumpulkan di satu objek konstanta `K` di dalam `index.html`, sehingga bisa diganti tanpa menyentuh logika.

**Tambahan teknis.** Grafik digambar dengan Canvas (tanpa pustaka chart), tema siang/malam untuk ruang kendali 24 jam, tata letak responsif sampai lebar ponsel, navigasi keyboard (`1`–`6` berpindah halaman, `Spasi` jeda), dan penghormatan terhadap `prefers-reduced-motion`.

---

## Isi dashboard

| Halaman | Menjawab pertanyaan |
|---|---|
| Executive overview | Apakah intake aman, apakah sampah menumpuk, apa yang harus dikerjakan shift ini |
| Live intake monitor | Berapa banyak puing mencapai rack dan kapan pembersihan harus dimulai |
| Waste management | Apa yang diangkat hari ini, ke mana perginya, berapa yang tetap jadi residu |
| Equipment & automation | Status peralatan, mode kendali, interlock, kesehatan pemeliharaan |
| Energy & carbon | Berapa energi yang dipakai sistem sampah dan berapa karbon yang dihindari |
| Business case | Telemetri yang sama dibaca sebagai uang: ketersediaan, penjualan material, payback |

## Kontrol interaktif

- **Skenario:** Normal operation, Rainfall spike, Waste surge, Trash rack loading
- **Pause / Resume**, kecepatan **1× / 2× / 4×**
- **AUTO / MANUAL**, Start–Stop conveyor, Run cleaning cycle, Steepen boom, Flag inspection, Acknowledge
- **Digital twin:** empat *slider* (intensitas hujan, efisiensi boom, kapasitas conveyor, efisiensi sorting) dengan proyeksi terhadap kondisi *live*, dan tombol **Apply to live simulation**
- **Ekspor:** telemetri CSV, material log CSV, energy CSV, audit trail CSV, dan *snapshot* JSON

---

## Ambang batas dan asumsi

| Parameter | Nilai | Letak |
|---|---|---|
| Pemicu pembersihan rack | 18 mbar | `K.DP_CLEAN` |
| Batas throttle intake | 28 mbar | `K.DP_THROTTLE` |
| Kapasitas collection cage | 240 kg | `K.CAGE_KG` |
| Level dispatch conveyor | 80% | `K.CAGE_DISPATCH` |
| Risiko luapan cage | 95% | `K.CAGE_CRIT` |
| Faktor emisi jaringan | 0,85 kg CO₂e/kWh | `K.GRID_EF` |
| Faktor landfill | 1,02 kg CO₂e/kg | `K.LANDFILL_EF` |
| Tarif listrik | Rp 1.420/kWh | `K.TARIFF` |
| CAPEX / OPEX | Rp 1,85 bn / Rp 186 jt per tahun | `K.CAPEX`, `K.OPEX` |

**Semua nilai di atas adalah nilai demonstrasi.** Begitu pula komposisi material, efisiensi sorting, specific energy consumption, dan daftar mitra pemulihan. Angka-angka ini dipilih agar logika kendali dapat ditunjukkan secara utuh, bukan sebagai hasil pengukuran. Sebelum dipakai secara operasional atau dikutip sebagai hasil, ganti dengan data lapangan: kurva ΔP terukur pada rack, timbangan cage, meteran energi auxiliary, hasil timbang bale per aliran material, dan faktor emisi jaringan setempat yang berlaku.

Model transduksi energi dan neraca massa di dalamnya disederhanakan secara sengaja. Dua hal yang paling perlu divalidasi lebih dulu: laju kenaikan ΔP terhadap massa puing (di sini linier, di lapangan bergantung pada jenis puing dan pola *matting*), serta hubungan kecepatan aliran dengan efisiensi intersepsi boom.

---

## Menyesuaikan untuk lokasi lain

Buka `index.html` dan ubah tiga tempat:

1. Objek `K` — seluruh ambang batas, kapasitas, faktor emisi, dan parameter ekonomi.
2. Objek `SCENARIOS` — karakter hidrologi tiap skenario (curah hujan dan pengali beban sampah).
3. Array `S.equip`, `PARTNERS`, dan `MATERIALS` — daftar peralatan, mitra pemulihan, dan komposisi material.

Nama plant ada di elemen `#plantName` beserta baris deskripsi di bawahnya.
