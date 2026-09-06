# Evidence Manifest — Chain of Custody

All evidence was captured 2026-09-04/05 from CP PLUS public endpoints. Toolchain: `curl`, `bsdtar`, `innoextract`, `7z`, `strings`, `pdftotext`, `openssl`, `iconv`, `shasum`. CAPTURED = directly verifiable from the saved file (hashed below). ASSERTED = requires re-capture.

> **Repo policy:** only code + technical proof is retained. Raw website HTML page dumps (marketing/product pages) were removed after the relevant text was extracted; the surviving artifacts below are the actual technical evidence the findings quote from.

---

## 1. Raw vendor downloads (hashed — see `MANIFEST_HASHES.txt`)

Paths are repo-root-relative. Every object is SHA-256-verifiable.

| Object | SHA-256 (first 16) | What it proves |
|---|---|---|
| `evidence/software/GPSCarKamPlayer.rar` | `cdb356b145f0f662…` | plain-HTTP Baidu map scripts in WebView2, UAC `requireAdministrator`, 2 hardcoded Google Maps API keys, no upload/cloud functions |
| `evidence/software/ipfinder_v4.exe` | `e188e88987a89aab…` | cloud-upgrade engine, batch device password modify/reset, Dahua-internal SDK DLLs, hardcoded Dahua P2P endpoint, baked creds |
| `evidence/software/kvms_prolite_v1.exe` | `e2b8bf9550d503ad…` | VMS / Command-Centre client: 14 Solutions DLLs, 90 locale modules, HumanTrait (soft-biometric) skin |
| `evidence/device_docs/CP-UNR-408T2-AI_manual.pdf` | `85bebdc54dcdedf7…` | defaults: Wi-Fi password `12345678`, ports 37777/37778/80/443/554, Instaon + Auto-Register default-on, face-lib 20/15MB, similarity-threshold note, modeling-failure admission |
| `evidence/device_docs/CP-UNR-408T2J-BI_manual.pdf` | `c07c999e0d4074a4…` | second MNVR manual (same default surface) |
| `evidence/device_docs/CP-UNR-404T1-AI_datasheet.pdf` | `ce415377eb8f2a71…` | spec sheet |
| `evidence/device_docs/CP-UNR-408T2-AI_datasheet.pdf` | `e9eba04e20865903…` | 8-ch PoE H.265 AI MNVR spec |
| `evidence/advisories/SA-202608-001.pdf` | `8011ee9874b3dd3d…` | vendor's own advisory: EZ-P21 — ACE via SD-card + unauthenticated snapshot/video endpoints (High) |
| `evidence/advisories/SA-CVSSv4-V1.pdf` | `1c0f54fddf01db99…` | vendor's own advisory: CP-UNR-104F1/108F1 stored XSS in NVR web UI (CVSS v4.0 6.8) — same UNR family as Ikshana MNVR |

## 2. Text extractions (`evidence/probe/`)

| File | What it proves |
|---|---|
| `manual_408T2-AI.txt` | full text of primary MNVR manual — every quote pair, with line numbers |
| `manual_408T2J-BI.txt` | full text of second MNVR manual |
| `ds_404T1-AI.txt` / `ds_408T2-AI.txt` | datasheet text extractions |
| `primary_headers.txt` | apex HTTP headers — no security headers, cookie missing `Secure` |
| `dmarc_check.txt` | empty `_dmarc` answer (no DMARC) |
| `firmware_api_all.json` | `/Firmware/GetByFilter` returns "No data Found" ⇒ no public firmware available (limits) |
| `software_files.txt` | the 16 public PC-tool download URLs from `/software` |

## 3. DNS / infra notes (asserted, re-capturable)

- Apex resolves to `209.38.127.6` (Azure), nginx/1.24.0 + ASP.NET Core, Azure DNS, M365 MX, SPF `-all`, no DMARC.

## 4. Extracted software trees (`evidence/software/`)

| Path | Contents |
|---|---|
| `carkam_extracted/` (569 files) | CarKam Player unpacked — `app/BMap.html` (plain-HTTP Baidu), `app/gmap.html` (API keys), WebView2 bundle, VLC plugins |
| `ipfinder_ext/` (546 files) | IP Finder unpacked — `dhlog.dll`, `DHDecode.dll`, `netsdk.dll`, `RootCert.cer` (C=CN), `public.pem` (1024-bit), batch templates, `Config/Weak_password_rules.txt`, `Config/Templet.xml`, `Config/deviceclass.xml` |
| `kvms_ext/` (3016 files) | KVMS Pro Lite unpacked — `Solutions/` (14 DLLs), `Languages/` (90 .qm), `Skin/DustyBlue/RealMonitorUI/HumanTrait/HumanAttributes/*.png` (soft-biometric icons) |

## 5. Derived analysis documents (`docs/`)

| File | Contents |
|---|---|
| `docs/AUDIT_REPORT.md` | 23 sections, 18 findings, claim-vs-reality register, limitations |
| `docs/ARTICLE.md` | publishable technical article (15 sections / 27 subsections) |
| `docs/ARTICLE_PLAIN.md` | plain-language article with inline glossary |
| `docs/DISCLOSURE_DRAFT.md` | responsible-disclosure draft routed to `/security-advisories` |
| `docs/CLAIM_vs_CODE.md` | claim → code reality table (CarKam no-upload ✅, IP Finder batch ops ✅, KVMS HumanTrait UI ✅) |
| `docs/proofs/PROOF_ChinaTech.md` | 7-point reproducible China-OEM proof |
| `docs/proofs/PROOF_FacialBroken.md` | 6-point vendor-manual-verified proof that "facial AI" is tunable/local/soft-biometric |
| `docs/product/BatchOps_Verified.md` | batch templates — saveConfig/SSHD enable, duress columns, SIP `123456`, baked creds |
| `docs/product/DeviceClass_Registry.md` | parsed `deviceclass.xml` (ACS/ARC/VDP/ANDROID/full camera classes) |
| `docs/product/Device_Feature_Dictionary.txt` | OEM feature strings (InstaOn Only, CMOB, templates) |
| `docs/product/KVMS_Solutions_inventory.txt` | KVMS `Solutions/` DLL catalog (14 files) |
| `docs/product/KVMS_module_inventory.txt` | KVMS `Languages/` .qm feature modules (90 live in 8 bundles) |
| `docs/product/KVMS_Skin_FeatureMap.txt` | HumanTrait icon inventory (sex/hat/bag/clothing) — the soft-biometric UI map |
| `docs/product/public_claim_linkedin.txt` | CP PLUS's public Ikshana claim text (the baseline claim checked) |

## 6. Scope notes

- No live-device interaction of any kind (no MNVR probing).
- No brute-force; `/Admin` was only fetched as a page, not authenticated against.
- "Only AI by Device" / modeling-failure / similarity-threshold quotes are verbatim from the vendor manual and are locatable via `grep -n` in `evidence/probe/manual_408T2-AI.txt`.
- Firmware reverse-engineering is out of scope because `/Firmware/GetByFilter` exposes no public firmware.