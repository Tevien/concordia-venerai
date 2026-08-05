# RAM RMA dossier — Corsair Vengeance LPX 64GB kit (stick B defect)

Everything needed to file and track the warranty return. Status also on the
project board ("GPU box hardware"); decision + trust rules in decisions.md
(2026-07-14 entry).

## Status

- **2026-07-14** — defect isolated to stick B by per-module testing; stick A
  clean. Box runs on stick A alone (32 GB) since.
- **Pending (user action):** file the RMA with Corsair, ship the kit, re-test
  both incoming sticks before trusting builds.

## The facts

| | |
|---|---|
| Kit | Corsair Vengeance LPX 64 GB (2×32 GB) DDR4-3200 CL16 |
| Part number | CMK64GX4M2E3200C16 |
| Serial / version | on the sticker of the **defective** module (Corsair asks for the "ver x.xx" + lot number printed there; dmidecode reports no serial) |
| Board | ASUS Prime B450, DDR4 at JEDEC 2133 MT/s — **no XMP/DOCP**, so well under the rated 3200 |
| OS during test | Ubuntu 24.04 |
| Bought | Azerty (NL), ~May 2025. Azerty's 1-year business warranty has lapsed → go **direct to Corsair** (lifetime warranty on memory) |
| Proof of purchase | Azerty invoice / order confirmation for the kit — have it ready |

## Symptoms (what it did in production)

Months of "storage" flakiness that was actually memory: dockerd deaths, CRC
errors on docker pulls (unpigz/tar), five different sha256 readings of one
file, and **3 of 12 model weight files silently corrupt** vs their HF
checksums. Everything that transited the box before 2026-07-14 is
checksum-verified before trust (standing rule).

## Diagnosis (why it's conclusively stick B)

Per-module isolation: each stick tested **alone, in the same DIMM slot**, with
a pattern test — fill 22 GB with deterministic data, verify in three rounds.

- **Stick A:** three full rounds, zero errors.
- **Stick B:** corruption in **all three rounds at the same address region**
  (chunks 22–23; six corrupt reads total) — a consistent defective DRAM
  region, not overclock instability (the kit never ran above JEDEC 2133).

## RMA form text (Dutch, ready to paste)

> Eén van de twee modules uit deze 64GB-kit (2×32GB, artikelnummer
> CMK64GX4M2E3200C16) veroorzaakt reproduceerbare, stille geheugencorruptie.
> Het geheugen draait op de standaard JEDEC-snelheid van 2133 MT/s — dus rúim
> onder de gespecificeerde 3200 MT/s, zonder XMP/DOCP-profiel — op een ASUS
> Prime B450-moederbord.
>
> De fout is per module geïsoleerd: beide modules zijn afzonderlijk, in
> hetzelfde geheugenslot, getest met een patroontest (22 GB vullen met
> deterministische data en in drie rondes verifiëren). Module A doorstond drie
> volledige rondes zonder één fout. Module B gaf in álle drie de rondes
> corruptie op exact hetzelfde adresgebied (zes corrupte leesacties in totaal)
> — een consistent defect gebied in het DRAM, geen instabiliteit door
> overklokken.
>
> In de praktijk leidde dit tot beschadigde downloads, checksum-fouten
> (CRC-mismatches) en vastlopers. Ik verzoek om vervanging van de defecte
> module onder de levenslange garantie van Corsair. Een memtest86+-rapport met
> fysieke adressen kan ik op verzoek aanleveren.

## The same, in English (Corsair's ticket portal is usually English)

> One of the two modules in this 64GB kit (2×32GB, part number
> CMK64GX4M2E3200C16) causes reproducible silent memory corruption. The
> memory runs at the default JEDEC speed of 2133 MT/s — well under the rated
> 3200, no XMP/DOCP profile — on an ASUS Prime B450 board.
>
> The fault is isolated per module: each module was tested alone, in the same
> DIMM slot, with a pattern test (fill 22 GB with deterministic data, verify
> in three passes). Module A passed three full passes with zero errors.
> Module B produced corruption in all three passes at exactly the same
> address region (six corrupt reads in total) — a consistent defective DRAM
> region, not overclock instability.
>
> In practice this caused corrupted downloads, checksum (CRC) mismatches and
> crashes. I request replacement of the defective module under Corsair's
> lifetime warranty. A memtest86+ report with physical addresses is available
> on request.

## Practical notes

- Corsair sells memory as a **kit** and normally RMAs the **full kit** — they
  may ask for both modules. Plan the downtime: the box would run without this
  memory while the kit is away. You can state in the form that module A tests
  clean and ask whether one module suffices, but expect a full-kit answer.
- Schedule shipment around render batches (the box is the render engine).
- Corsair's flow: corsair.com → Support → create RMA ticket under the account;
  photos of the module sticker (part no. + ver/lot) speed it up.

## When the replacement kit arrives

1. memtest **both** sticks (same per-module method, 3 passes) before any
   trusted build — the standing rule from decisions.md stays until this
   passes.
2. Restore dual-channel (2×32 GB), confirm 2133 JEDEC (or enable DOCP only
   after a clean full-kit pass at 3200 if desired).
3. Re-run a checksum spot-check on fresh downloads, then lift the
   "no trusted builds" flag in the memory file + board.
