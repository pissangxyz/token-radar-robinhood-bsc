# Token Radar — Robinhood Chain + BSC (Free)

Skill radar token gratis dan berkualitas untuk dua jaringan: **Robinhood Chain** (L2 baru, mainnet Juli 2026) dan **BNB Smart Chain / BSC** (jaringan meme/DeFi yang ramai). Semua sumber data di skill ini GRATIS tanpa API key. Dipakai untuk: nemu token baru, nilai kualitasnya, dan kasih vonis beli/jangan — BUKAN saran finansial, dan JANGAN pernah beli/swap tanpa perintah eksplisit user.

**Dibangun:** 2026-11-23, atas permintaan user ("bikinin aku skill radar token jaringan robin sama bsc free berkualitas").
**Semua endpoint di bawah DIVERIFIKASI HIDUP pada 2026-11-23 16:00 WIB** (cek langsung ke API-nya, bukan dari dokumentasi saja).

## 1. Fakta jaringan

| | Robinhood Chain | BSC |
|---|---|---|
| Chain ID (EIP-155) | **4663** (testnet 46630) | **56** |
| Jenis | Arbitrum Orbit L2, mainnet 1 Juli 2026 | L1 EVM |
| Gas | ETH | BNB |
| RPC publik (rate-limited) | `https://rpc.mainnet.chain.robinhood.com` | `https://bsc-dataseed.binance.org` |
| Explorer | `https://robinhoodchain.blockscout.com` (Blockscout, API ala Etherscan, tanpa key) | `https://bscscan.com` |
| DEX utama | Uniswap v3/v4 (live sejak hari pertama), Pons (launchpad) | PancakeSwap v2/v3 |
| Token khas | Saham tokenisasi (GME, AAPL, NVDA…), USDG, WETH `0x0Bd7D308f8E1639FAb988df18A8011f41EAcAD73` | CAKE, WBNB `0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c` |
| Karakter | MUDA & TIPIS: likuiditas kecil, pool baru tiap menit, banyak token iseng | RAMAI & MATANG: likuiditas besar, tapi honeypot/rug juga paling banyak |

## 2. Sumber data gratis (tanpa key)

| Sumber | Base URL / endpoint | Rate limit | Dipakai untuk | Catatan |
|---|---|---|---|---|
| **DexScreener** | `https://api.dexscreener.com` | ~300 req/menit (umum), 60/menit utk profile/boost | Harga, likuiditas, volume, txns, FDV, umur pair, cari pair/token | **Slug Robinhood Chain = `robinhood`, BUKAN `4663`.** Pakai `4663` hasilnya kosong diam-diam (bukan error). Slug BSC = `bsc`. |
| **GeckoTerminal** | `https://api.geckoterminal.com/api/v2` | ~30 req/menit (free) | Pool baru, trending, OHLCV, trades | Network slug: `robinhood` dan `bsc`. Endpoint kunci: `/networks/{network}/new_pools`, `/networks/{network}/trending_pools`, `/networks/{network}/tokens/{address}` |
| **GoPlus Token Security** | `https://api.gopluslabs.io/api/v1/token_security/{chain_id}?contract_addresses={addr}` | ringan, tanpa key | Honeypot, buy/sell tax, mintable, proxy, blacklist, hidden owner, owner powers, holder top-10 + `is_locked` | **Terverifikasi dukung chain 4663 dan 56.** Satu call bisa 30 alamat (pisah koma). |
| **honeypot.is** | `https://api.honeypot.is/v2/IsHoneypot?address={addr}&chainID={id}` | tanpa key | Simulasi buy lalu sell beneran: apakah bisa jual, tax yang terukur, gas | **Hanya ETH (1), BSC (56), Base (8453). TIDAK dukung Robinhood Chain.** Untuk Robinhood, ganti dengan simulasi baca pool + GoPlus. |
| **Blockscout (Robinhood)** | `https://robinhoodchain.blockscout.com/api` | tanpa key | Verifikasi kontrak, holder, transaksi, umur token, baca source | Endpoint ala Etherscan: `?module=token&action=getTokenHolders`, `?module=contract&action=getsourcecode` |
| **BscScan** | `https://api.bscscan.com/api` | free tier 5 req/detik (tanpa key makin ketat) | Verifikasi kontrak, holder, transaksi BSC | Kalau kena limit, pakai GoPlus + DexScreener dulu |

### Endpoint contoh (sudah dites hidup)

```bash
# Pool baru di Robinhood Chain (GeckoTerminal) — token-token yang baru lahir
curl -s "https://api.geckoterminal.com/api/v2/networks/robinhood/new_pools"

# Pool baru di BSC
curl -s "https://api.geckoterminal.com/api/v2/networks/bsc/new_pools"

# Semua pool untuk satu token di Robinhood (DexScreener)
curl -s "https://api.dexscreener.com/token-pairs/v1/robinhood/0x5fc5360D0400a0Fd4f2af552ADD042D716F1d168"

# Semua pool untuk satu token di BSC
curl -s "https://api.dexscreener.com/token-pairs/v1/bsc/0x0e09fabb73bd3ade0a17ecc321fd13a19e81ce82"

# Cari pair/token (DexScreener, semua chain)
curl -s "https://api.dexscreener.com/latest/dex/search?q=USDG"

# Keamanan kontrak — Robinhood (4663) dan BSC (56)
curl -s "https://api.gopluslabs.io/api/v1/token_security/4663?contract_addresses=0x5fc5360D0400a0Fd4f2af552ADD042D716F1d168"
curl -s "https://api.gopluslabs.io/api/v1/token_security/56?contract_addresses=0x0e09fabb73bd3ade0a17ecc321fd13a19e81ce82"

# Simulasi honeypot — BSC saja
curl -s "https://api.honeypot.is/v2/IsHoneypot?address=0x0e09fabb73bd3ade0a17ecc321fd13a19e81ce82&chainID=56"
```

**Aturan panggil API:** selalu kirim `User-Agent` yang jelas, hormati rate limit (jeda antar call), dan kalau satu sumber jawab kosong/error, JANGAN anggap tokennya aman/berbahaya — pindah ke sumber lain. DexScreener jawab `[]`/`null` untuk token yang belum terdaftar DAN untuk network yang salah — bedakan keduanya dengan cek slug dulu.

## 3. Workflow radar (4 langkah, selalu urut)

### Langkah 1 — Discovery (nemu token baru)
- **Robinhood:** `GeckoTerminal /networks/robinhood/new_pools` + `trending_pools`, dan `dexscreener.com/robinhood` (feed new pairs). Ambil 10–20 pool terbaru.
- **BSC:** `GeckoTerminal /networks/bsc/new_pools` + `trending_pools`, dan `dexscreener.com/bsc`.
- Saring awal: buang pool dengan `reserve_in_usd` = 0, `volume_usd` h24 = 0, atau `fdv_usd` absurd (mis. $0.00000004 — itu token mati/spam, bukan peluang).
- Catat untuk tiap kandidat: alamat kontrak token (BUKAN alamat pair), nama, simbol, DEX, umur pool, FDV, likuiditas, volume h24, txns h24 (buys/sells, buyers/sellers).

### Langkah 2 — Market screen (apakah tokennya "hidup"?)
Dari data DexScreener/GeckoTerminal, nilai:
- **Likuiditas:** makin besar makin susah di-rug via tarik likuiditas. Ambang kasar: Robinhood ≥ $10k (rantai muda, standar lebih rendah), BSC ≥ $50k.
- **Volume h24 vs likuiditas:** volume sehat = ada perdagangan beneran, bukan cuma 1–2 wallet bolak-balik. Waspada kalau volume h24 > 10× likuiditas tapi buyer unik sedikit — itu pola wash.
- **Trader unik:** buys dan sells seimbang + buyers ≥ 20 dalam 24 jam = minat menyebar. Kalau buys 100 tapi buyers cuma 3, itu 3 wallet yang sama.
- **Umur pool:** < 1 jam = lotre murni (paling rawan rug); 1–24 jam = pantau ketat; > 7 hari + likuiditas stabil = lebih teruji.
- **Harga vs FDV:** FDV gede + likuiditas kecil = gampang dimanipulasi. Bandingkan FDV dengan market cap kalau ada (GeckoTerminal kadang isi `market_cap_usd`).

### Langkah 3 — Safety screen (apakah kontraknya jahat?)
Jalankan untuk SETIAP kandidat yang lolos langkah 2 — jangan pernah skip:
1. **GoPlus** (kedua chain): baca `is_honeypot`, `cannot_buy`, `buy_tax`, `sell_tax`, `transfer_tax`, `is_mintable`, `is_proxy`, `is_blacklisted`, `hidden_owner`, `owner_change_balance`, `slippage_modifiable`, `selfdestruct`, `trading_cooldown`, `is_open_source`, dan top holders (`percent`, `is_locked`).
2. **honeypot.is** (BSC saja): `isHoneypot` harus false, `risk` low, buy/sell tax 0 atau kecil, `simulationSuccess` true.
3. **Holder & likuiditas:** top-1 holder (di luar dead/burn address) > 20% = konsentrasi bahaya; LP holder ter-lock (`is_locked: 1` di GoPlus) = likuiditas terkunci, bagus; cek apakah LP token dipegang deployer.
4. **Verifikasi kontrak:** `is_open_source` = 1. Kontrak unverified = jangan sentuh, tanpa kecuali.
5. **Deployer:** cek alamat creator di explorer — kalau deployer yang sama pernah bikin honeypot lain (`honeypot_with_same_creator` = 1 di GoPlus), langsung buang.

### Langkah 4 — Vonis (skor 0–100 + label)
Hitung skor dari 4 komponen (bobot):

| Komponen | Bobot | Yang dinilai |
|---|---|---|
| Keamanan kontrak | 40 | GoPlus + honeypot.is: honeypot/tax/owner/proxy/mint |
| Likuiditas & lock | 25 | Besaran likuiditas, LP terkunci, umur pool |
| Aktivitas pasar | 20 | Volume h24, trader unik, seimbang buy/sell |
| Distribusi holder | 15 | Holder count, konsentrasi top holder, creator % |

**Label vonis:**
- **SAFE (75–100):** lolos semua hard check, likuiditas + aktivitas sehat. Boleh masuk watchlist. Tetap BUKAN ajakan beli.
- **CAUTION (50–74):** ada satu-dua kuning (mis. tax kecil, mintable tapi owner renounced/terpercaya, holder agak terkonsentrasi). Pantau dulu, jangan kejar.
- **DANGER (0–49):** kena hard check mana pun (di bawah). Jangan sentuh.

**Hard check = langsung DANGER, tanpa peduli skor:**
- `is_honeypot` = 1 atau honeypot.is bilang honeypot
- `cannot_buy` = 1 atau sell tax ≥ 50% (atau sell tax tidak terukur karena simulasi gagal)
- `is_blacklisted` = 1, `selfdestruct` = 1, `hidden_owner` = 1
- Kontrak tidak terverifikasi (`is_open_source` = 0)
- Likuiditas 0 / pool mati / `reserve_in_usd` = 0
- Deployer pernah bikin honeypot (`honeypot_with_same_creator` = 1)

**Format laporan per token (selalu pakai ini):**
```
Token: NAMA (SIMBOL) — Chain
Kontrak: 0x...
Pair: DEX + alamat pair | Umur: ... | FDV: $... | Likuiditas: $... | Vol h24: $...
Trader h24: X buys / Y sells, Z buyer unik
GoPlus: honeypot ..., buy tax ...%, sell tax ...%, mintable ..., proxy ..., owner ...
Honeypot.is (BSC): risk ..., isHoneypot ...
Holder: N holder, top-1 ...% (locked? ...)
Vonis: SAFE/CAUTION/DANGER (skor X/100) — alasan satu kalimat
```

## 4. Jebakan khusus tiap jaringan

### Robinhood Chain — rantai muda, jebakan khas:
- **Token kembar/imitasi:** dua token berbeda bisa pakai simbol sama. TERBUKTI: ada dua token `USDG` — yang asli Global Dollar `0x5fc5360D0400a0Fd4f2af552ADD042D716F1d168` dan satu lagi `0x8218d73C00567A01481495Ad6c5143e00D5BB5b4` ("Useless Stupid Degen Gamblers"). **Selalu verifikasi lewat alamat kontrak, JANGAN lewat nama/simbol.**
- **Likuiditas tipis = slippage brutal:** pool $1k–$20k itu normal di sini. Order kecil saja bisa gerakin harga 10%+. Jangan samakan standar BSC.
- **Token saham (GME, AAPL, NVDA…):** itu tokenisasi resmi, bukan meme — nilainya ngikut harga saham aslinya. Jangan nilai pakai rubrik meme (FDV/volumenya beda dunia).
- **Launchpad Pons/Clanker:** banyak token lahir dari sini. Token launchpad = umur hitungan menit, ekspektasi rug tinggi; terapkan hard check tanpa kompromi.
- **honeypot.is TIDAK dukung chain ini** — jangan klaim hasil simulasi honeypot untuk Robinhood; pakai GoPlus + baca pool (reserve, tax dari GoPlus) + cek manual sell-ability via simulasi baca kontrak kalau perlu.
- **RPC publik rate-limited** — untuk pemakaian berat, catat bahwa user butuh provider (Alchemy/QuickNode/dRPC) — tapi skill ini tidak butuh RPC sama sekali untuk radar dasar.

### BSC — rantai ramai, jebakan khas:
- **Honeypot paling banyak di sini.** Jangan pernah skip honeypot.is + GoPlus, walau tokennya kelihatan "ramai".
- **Tax tersembunyi:** buy tax 0 tapi sell tax 99% itu pola klasik. Selalu baca KEDUA tax.
- **Wash trading:** volume gede + trader unik dikit = manipulasi. Selalu cek `buyers`/`sellers` unik, bukan cuma jumlah transaksi.
- **"Mintable" bukan otomatis jahat:** CAKE sendiri `is_mintable: 1` karena desain tokenomics-nya. Bedakan mintable-dengan-owner-aktif (bahaya) vs mintable-dengan-owner-renounced/terpercaya (konteks).

## 5. Batasan jujur (jangan pernah langgar)

- **Ini BUKAN saran finansial.** Skor dan vonis adalah penilaian risiko point-in-time, bukan prediksi harga. Token SAFE pun bisa turun 90%.
- **Hasil cek kedaluwarsa cepat.** Owner bisa ganti tax, buka blacklist, atau tarik likuiditas SETELAH cek. Vonis hanya berlaku untuk saat dicek — sebutkan jam ceknya di laporan.
- **Tidak ada cek yang 100%.** GoPlus dan honeypot.is masing-masing punya blind spot; itu kenapa skill ini pakai minimal 2 sumber keamanan + data pasar.
- **Jangan pernah beli, swap, approve, atau pindahin aset atas nama user** tanpa perintah eksplisit user untuk transaksi itu, dengan preview dulu (aturan wallet yang berlaku).
- **Jangan pakai sumber berbayar** (Bitquery, Birdeye Pro, Nansen, dsb.) tanpa persetujuan user — skill ini gratis penuh.
- **Jangan percaya nama/simbol token.** Selalu alamat kontrak. Selalu.
- Kalau API berubah (slug baru, endpoint mati, rate limit berubah), catat di skill ini dan sesuaikan — jangan ngarang endpoint.

## 6. Cara pakai

- "Radar token baru di Robinhood" → Langkah 1–4 untuk chain Robinhood, laporkan top 5 kandidat + vonisnya.
- "Cek token 0x... di BSC" → Langkah 2–4 untuk alamat itu, satu laporan lengkap.
- "Pantau token X" → ulang Langkah 2–3 berkala; laporkan HANYA kalau vonis berubah atau ada red flag baru (jangan spam).
- "Bandingin token A vs B" → jalankan Langkah 2–4 untuk keduanya, sajikan berdampingan.
- **"Sniper token baru" → pakai Mode Sniper di bawah (aturan longgar, target koin baru, jual otomatis saat untung).**

## 6b. MODE SNIPER v2 (aktif — perintah user 2026-11-23 16:28 WIB)

> "Coba tanpa GoPlus, tapi tetap gunakan saat token dibeli — kalau tokennya bermasalah jangan disentuh lagi. Tambah juga auto-sell kalau dev jual dan likuiditas ditarik."

Revisi dari v1 berdasarkan pelajaran simulasi: GoPlus terlalu lambat di menit ke-0 (6/8 token belum terindeks bahkan setelah 15 menit) dan gate "kontrak unverified" malah nge-block pemenang (AI +104%, Shit +85%). Jadi v2 membalik urutannya: **masuk dulu dengan gate ringan, verifikasi jalan terus SETELAH beli — dan kalau token ketahuan bermasalah, jual + blacklist selamanya.**

### Target buruan
- Pool umur **0–60 menit** yang baru muncul di `new_pools` — makin baru makin bagus.

### Gate masuk v2 — TANPA GoPlus, TANPA status verifikasi
Masuk kalau SEMUA ini terpenuhi (cek via DexScreener/GeckoTerminal saja — cepat, tanpa tunggu indeks):
1. Pool ada dan **likuiditas (reserve) > 0**.
2. **Ada buys yang terjadi** (bukan pool mati — minimal 1 buy).
3. Harga masih bisa dibaca (pair aktif di DexScreener).

**Yang TIDAK lagi jadi gate masuk:** GoPlus (belum terindeks = bukan alasan menunda), status verifikasi kontrak, likuiditas minimum, FDV, volume minimum, trader unik, konsentrasi holder, tax. Semua itu pindah jadi bahan monitoring pasca-beli.

### Monitoring pasca-beli — GoPlus tetap dipakai DI SINI
Begitu posisi masuk, jalanin loop ini terus sampai posisi keluar (polling tiap 1–5 menit, hormati rate limit: DexScreener ~300/menit, GeckoTerminal ~30/menit):
1. **GoPlus re-check:** begitu GoPlus sudah mengindeks token, baca hasilnya. Kalau `is_honeypot = 1`, `cannot_buy = 1`, `cannot_sell_all = 1`, sell tax ≥ 50%, `selfdestruct = 1`, `hidden_owner = 1`, atau `is_blacklisted = 1` → **auto-sell segera + masukkan ke blacklist (jangan pernah sentuh lagi).**
2. **Dev-sell watch:** pantau dompet dev/creator (alamat creator dari GoPlus saat sudah terindeks, atau dari transaksi pembuatan pair). Kalau dev menjual — ambang: **penjualan berapa pun dalam 24 jam pertama, atau > 10% dari pegangan dev** → **auto-sell segera + blacklist.** Cara pantau gratis: cek berkala `creator_balance`/`creator_percent` di GoPlus, atau daftar transfer token dari alamat creator di explorer. Jujur: deteksi ini punya jeda polling (hitungan menit) — dev bisa dump duluan sebelum cek berikutnya jalan. Ini mitigasi, bukan jaminan.
3. **Liquidity-pull watch:** pantau `liquidity.usd` (DexScreener) / `reserve_in_usd` (GeckoTerminal) tiap polling. Kalau likuiditas turun **> 50% dalam 5 menit** atau jatuh di bawah 50% dari likuiditas saat entry → **auto-sell segera.** Jujur: kalau likuiditas ditarik TOTAL sekaligus (kasus 金狗: $141k → $0), tidak ada yang bisa dijual — trigger ini hanya menyelamatkan saat penarikan bertahap/parsial.
4. **Blacklist permanen:** token yang kena flag pasca-beli (honeypot, kontrak berbahaya, dev rug, LP pull total) dicatat di `blacklist.md` (folder skill ini) dengan alamat, chain, tanggal, dan alasan. **Token di blacklist tidak pernah di-entry lagi, apa pun yang terjadi.** Cek blacklist SEBELUM setiap entry baru.

### Jual otomatis (take profit + proteksi) — v2
- **TP1 +100% (2x): jual 50%. TP2 +200% (3x): jual 25% lagi. Sisa 25%: trailing stop −30% dari puncak.**
- **Stop-loss (CL / cut-loss) −60% dari entry — diubah dari −50% atas perintah user 2026-11-23 17:44 WIB ("dari 50% jadi 60/70").** Max hold 24 jam. (Catatan: −70% tinggal bilang, satu kata — makin longgar makin dalam luka per token yang gagal, tapi makin besar ruang buat token yang sempat turun lalu balik naik.)
- **BARU — auto-sell pemicu darurat (jual semua, segera):** GoPlus pasca-beli flag bahaya; dev jual (ambang di atas); likuiditas turun > 50% dalam 5 menit.
- Urutan prioritas saat beberapa pemicu kena bersamaan: pemicu darurat > stop-loss > TP.
- Slippage: catat toleransi sebelum masuk (pool tipis → slippage besar itu normal).

### Auto-revoke (baru — perintah user 2026-11-23 16:31 WIB)
Setelah jual, jangan biarkan izin (approval) token menggantung — kontrak jahat bisa memakainya belakangan untuk menarik token lo.
- **Kapan revoke:** (a) setelah posisi **tertutup penuh** (100% terjual — TP2/trailing/SL/max-hold/pemicu darurat yang menghabiskan posisi); (b) **segera** untuk token yang masuk blacklist; (c) sapu berkala tiap 24 jam: revoke approval token yang sudah tidak dipegang lagi.
- **JANGAN revoke setelah jual parsial (TP1 50%)** — sisa posisi masih butuh approval untuk dijual nanti; revoke di situ cuma buang gas dan memaksa approve ulang.
- **Caranya:** kirim `approve(spender, 0)` ke kontrak token. Spender yang di-revoke = router yang dipakai buat trade itu: PancakeSwap Router (BSC); Uniswap Universal Router / **Permit2** (Robinhood Chain — Uniswap v4 memakai Permit2, jadi revoke ke Permit2).
- **Biaya:** 1 transaksi gas per revoke; murah di BSC dan Robinhood Chain (gas ETH recehan), tapi tetap dicatat sebagai biaya exit.
- **Batasan jujur:** revoke melindungi dari penyalahgunaan allowance yang tersisa. Ia **TIDAK** mengembalikan dana yang sudah hilang, **TIDAK** mencegah rug likuiditas, dan **TIDAK** memperbaiki honeypot (kalau tidak bisa jual, revoke tidak ada gunanya). Beberapa token punya `approve()` yang aneh (revert saat set 0) — kalau revoke gagal, catat dan coba sekali lagi; jangan dipaksa berulang kali.
- **Eksekusi:** revoke adalah transaksi on-chain — sama seperti trade, butuh perintah eksplisit user + preview sebelum dikirim.

### Position sizing
- Per token: **2% dari bankroll sniper** — standar (diubah dari rentang 2–5% atas perintah user 2026-11-23 17:51 WIB). Jangan all-in satu token. Alasan: rug adalah pembunuh utama (3 dari 16 token mati di dua simulasi) dan hanya ukuran posisi yang melindungi dari rug — CL/trailing tidak bisa menyelamatkan token yang likuiditasnya ditarik. Dua rug dalam satu ronde = −4% bankroll (bukan −10% kalau 5% per token).

### Batasan eksekusi (penting, jangan dilanggar)
- Skill ini mendefinisikan **aturan** sniper. Eksekusi live (bot + wallet + gas + dana) adalah langkah terpisah: butuh dana di wallet, dan **setiap transaksi tetap butuh preview + perintah eksplisit user** (aturan wallet yang berlaku). Jangan pernah klaim auto-trading sudah jalan kalau belum di-setup dan disetujui.
- **Fakta saat ini (2026-11-23): user belum ada dana** — mode sniper disiapkan dulu sebagai strategi; live sniping jalan setelah ada dana + perintah eksplisit untuk deploy.
- **Jujur soal v2: ini "beli dulu, verifikasi kemudian" — risikonya memang lebih tinggi dari v1, itu disengaja.** Mitigasinya adalah ukuran kecil + monitoring pasca-beli yang ketat + auto-sell darurat + blacklist permanen. Bukan saran finansial.

## 7. Pelajaran dari simulasi live (2026-11-23 16:11 WIB)

Simulasi pertama skill ini (8 token baru, 4 Robinhood + 4 BSC, umur 1–9 menit) membuktikan pola penting:

- **Token berumur menit hampir selalu BELUM terindeks alat keamanan.** GoPlus batch cuma mengembalikan token yang sudah terindeks — sisanya hilang diam-diam dari respons (bukan error). honeypot.is menjawab HTTP 404 `{"code":404,"error":"pair not found"}` untuk pair yang belum terindeks. Jadi: **hasil kosong/404 = "belum bisa dicek", BUKAN "aman".** Vonis maksimal untuk token yang safety screen-nya tidak jalan adalah CAUTION — jangan pernah naikkan ke SAFE.
- **Data GoPlus bisa basi untuk token sangat baru.** Contoh: CHOP (Robinhood) dilaporkan `creator_percent: 1.0` (creator pegang 100% supply) padahal `is_in_dex: 0` — GoPlus belum sinkron dengan pool-nya. Tetap vonis DANGER (flag ekstrem + tak terverifikasi), tapi tulis caveat-nya dan cek ulang beberapa jam kemudian.
- **Saringan discovery bekerja.** Token dengan 1 transaksi / volume ~$0 (STONKS, NIKUMARU, MEMEFORGE di Robinhood; token four-meme bervolume $0.001 di BSC) langsung terbuang di Langkah 1 — jangan buang waktu cek keamanan untuk token mati.
- **Hasil simulasi: 0 SAFE dari 8 kandidat, 1 DANGER (CHOP), 7 CAUTION.** Itu normal untuk pool berumur menit — radar ini memang dirancang untuk menolak memberkati yang belum terverifikasi, bukan untuk memaksa ada pemenang.

## 8. Pelajaran dari simulasi Mode Sniper (2026-11-23 16:19 WIB)

Simulasi sniper: 8 token yang sama dari simulasi pertama, dicek ulang ~8 menit setelah entry (~16:11 → ~16:19 WIB), pakai gate longgar (hanya anti-honeypot + kontrak berbahaya):

- **Masuk (6):** CHOP +74% (lolos GoPlus, creator 100% bukan gate), DELULU −21% (ABI ERC20 bersih, tanpa mint/pause/blacklist), SHORT +130% (**TP1 kena, jual 50%**), 金狗 −100% (**RUG, likuiditas $141k → $0**), STINGRAY +19%, MuskTRUMP +72% (keduanya UNVERIFIED — GoPlus belum indeks, ABI BscScan tidak bisa diambil tanpa API key).
- **Diblokir (2):** AI (+104%, TP1 kena — tapi kontrak **belum terverifikasi** → block) dan Shit (+85%, alasan sama). Gate keamanan membuang 2 peluang terbaik — trade-off nyata kecepatan vs keamanan.
- **Pelajaran keras:**
  1. Rug via tarik likuiditas TIDAK ketangkep gate sniper — 金狗 reserve terbesar ($141k) tetap rug 10 menit kemudian. LP lock sengaja bukan gate di mode ini; itu harga yang dibayar.
  2. Stop-loss −50% tidak berguna kalau likuiditas ditarik (tidak ada pembeli — tidak bisa jual).
  3. Verifikasi di menit ke-0 sering mustahil: GoPlus belum mengindeks 6/8 token bahkan setelah 15 menit; Blockscout API 403 via text fetch; BscScan tanpa API key mengalihkan ke halaman docs. Sniper sering masuk buta (UNVERIFIED).
  4. Harga quoted ≠ harga fill — slippage di pool $10–60k brutal; jual 50% posisi bisa menggerakkan harga sendiri.
  5. 0 sells = kemampuan jual belum terbukti (MuskTRUMP 26 buys / 0 sells).
  6. Jendela 8 menit terlalu pendek untuk vonis — ini snapshot, bukan backtest.
