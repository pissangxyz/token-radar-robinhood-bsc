# Token Radar — Robinhood Chain + BSC

Skill radar token **gratis** (tanpa API key) untuk dua jaringan:

- **Robinhood Chain** (chain ID 4663, Arbitrum Orbit L2)
- **BNB Smart Chain / BSC** (chain ID 56)

## Yang dilakuin skill ini

1. **Discovery** — nemu token/pool baru (GeckoTerminal new pools + DexScreener)
2. **Market screen** — likuiditas, volume, trader unik, umur pool
3. **Safety screen** — GoPlus (honeypot, tax, mint, proxy, owner) + honeypot.is untuk BSC (simulasi buy-sell beneran)
4. **Vonis** — skor 0–100, label SAFE / CAUTION / DANGER + hard-check list

## Catatan penting

- DexScreener pakai slug `robinhood`, **bukan** `4663`.
- Ada token kembar di Robinhood (dua token `USDG`) — selalu verifikasi lewat **alamat kontrak**, jangan lewat nama/simbol.
- honeypot.is **tidak** mendukung Robinhood Chain, hanya ETH/BSC/Base.
- Ini radar + penilaian risiko point-in-time, **bukan saran finansial**.

## File

- `SKILL.md` — skill lengkap (bahasa Indonesia)

Dibangun oleh Jojo untuk yangmuliapadukasultanfadhil, 2026-11-23. Semua endpoint diverifikasi hidup pada 2026-11-23 16:00 WIB.
