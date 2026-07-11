# Deploy: GitHub → Vercel → Google Apps Script

Repo ini adalah **wrapper** (3 file statis) yang membuat aplikasi Google Apps
Script bisa diakses lewat domain kustom (`namamu.vercel.app` atau domain
sendiri), lengkap dengan loading screen. Konten aslinya tetap dilayani oleh
Google (via iframe), jadi `google.script.run`, login, cart, dsb tetap
berfungsi 100% normal — tidak ada yang di-proxy/di-fetch ulang.

```
vercel-deploy/
├── index.html    ← halaman wrapper (iframe full-page + loading skeleton)
├── config.js     ← SATU-SATUNYA file yang perlu kamu edit
├── vercel.json   ← konfigurasi Vercel (cache, header)
└── README.md     ← panduan ini
```

## 1) Deploy Apps Script sebagai Web App

1. Buka project Apps Script kamu (`Code.gs` + `index.html` yang sudah ada).
2. **Deploy → New deployment**.
3. Pilih tipe **Web app**.
4. Isi:
   - **Execute as**: `Me` (akun kamu)
   - **Who has access**: `Anyone`
5. Klik **Deploy**, lalu **copy URL** yang diakhiri `/exec`.

> ✅ Kode `doGet()` di `Code.gs` kamu **sudah** berisi
> `.setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)` — artinya
> halaman ini memang sudah diizinkan untuk di-iframe dari domain lain
> (Vercel). Tidak ada yang perlu diubah di sisi GAS.

Setiap kali kamu deploy versi baru dari editor Apps Script (**Manage
deployments → Edit → New version**), URL `/exec` yang sama tetap berlaku,
jadi kamu **tidak perlu** update `config.js` lagi setelahnya — kecuali kamu
sengaja membuat deployment baru dari nol.

## 2) Push repo ini ke GitHub

```bash
git init
git add .
git commit -m "Setup Vercel wrapper untuk GAS web app"
git branch -M main
git remote add origin https://github.com/USERNAME/NAMA-REPO.git
git push -u origin main
```

## 3) Import ke Vercel

1. Buka [vercel.com/new](https://vercel.com/new) → **Import Git Repository**.
2. Pilih repo yang baru saja di-push.
3. Framework preset: pilih **Other** (situs ini murni HTML statis, tidak
   perlu build command / output directory khusus).
4. Klik **Deploy**.

Setelah deploy pertama selesai, kamu akan dapat URL default seperti
`nama-repo.vercel.app`.

## 4) Isi `config.js` dengan URL GAS kamu

Edit `config.js` di GitHub (langsung lewat web GitHub atau `git push` lagi):

```js
window.APP_CONFIG = {
  gasExecUrl: "https://script.google.com/macros/s/AKfycb.../exec",
  title: "Rumah Eduwisata Sembalun",
  loadingText: "Memuat Rumah Eduwisata Sembalun..."
};
```

Commit & push → Vercel otomatis redeploy dalam beberapa detik. Kalau
`gasExecUrl` belum diisi/salah format, halaman akan menampilkan pesan
peringatan alih-alih layar putih kosong (jadi gampang di-debug).

## 5) (Opsional) Pasang custom domain

Di dashboard Vercel project → **Settings → Domains** → tambahkan domain
kamu sendiri (mis. `rumaheduwisatasembalun.id`), lalu arahkan DNS sesuai
instruksi Vercel (biasanya CNAME atau A record). Domain kustom otomatis
dapat SSL gratis dari Vercel.

## Catatan teknis

- **Kenapa iframe, bukan reverse-proxy?** `google.script.run` (yang dipakai
  di seluruh aplikasi untuk komunikasi ke backend) hanya berfungsi kalau
  konten benar-benar dilayani langsung dari domain Google. Kalau HTML-nya
  di-fetch ulang lalu disajikan dari domain Vercel (reverse-proxy penuh),
  mekanisme sandbox itu akan patah dan seluruh fitur login/cart/dashboard
  berhenti berfungsi. Iframe menjaga konten tetap dilayani native oleh
  Google sambil domain di address bar tetap punya kamu.
- **Height 100dvh** dipakai di iframe supaya tidak ada "lompatan" tinggi
  saat address bar browser mobile muncul/hilang saat scroll.
- Kalau nanti butuh sinkronisasi tinggi/scroll antara wrapper dan konten
  (misalnya untuk animasi transisi tambahan), bisa ditambahkan komunikasi
  `postMessage` antara `index.html` (wrapper) dan halaman GAS — beri tahu
  saya kalau perlu, saya bisa tambahkan.
