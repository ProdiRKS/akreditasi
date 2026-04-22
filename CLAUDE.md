# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Konteks Repositori

Repositori ini berisi dokumen **Laporan Evaluasi Diri (LED)** untuk reakreditasi **Program Studi Rekayasa Keamanan Siber (PS RKS) D4** di Politeknik Siber dan Sandi Negara (Poltek SSN), menggunakan instrumen **LAM Infokom 2.1** (November 2025).

Semua dokumen LED ditulis dalam **Bahasa Indonesia**.

## Struktur Dokumen

### Kriteria LED (6 kriteria)
LED terdiri dari 6 kriteria (C.1–C.6), masing-masing mengikuti siklus **PPEPP** (5 tahap):

| Tahap | Kode file | Isi |
|-------|-----------|-----|
| Penetapan | `LED-CX.1-*.md` | Kebijakan, standar mutu, target indikator |
| Pelaksanaan | `LED-CX.2-*.md` | Data aktual dari LKPS |
| Evaluasi | `LED-CX.3-*.md` | Hasil AMI 2025; tercapai/KTS/Observasi |
| Pengendalian | `LED-CX.4-*.md` | Tindak lanjut temuan |
| Peningkatan | `LED-CX.5-*.md` | Target baru, strategi siklus berikutnya |

Setiap kriteria juga memiliki **file gabungan** (`LED-CX-*.md`) yang menggabungkan kelima tahap PPEPP dengan heading di-bump satu level (`#` → `##`, `##` → `###`, dst.).

### Status Pengerjaan
- **C.1 Budaya Mutu** — selesai (C1.1–C1.5 + gabungan `LED-C1-Budaya-Mutu.md`)
- **C.2 Relevansi Pendidikan** — selesai (C2.1–C2.5 + gabungan `LED-C2-Relevansi-Pendidikan.md`)
- **C.6 Diferensiasi Misi** — selesai (C6.1–C6.5 + gabungan `LED-C6-Diferensiasi-Misi.md`)
- **C.3, C.4, C.5** — belum dibuat

### Sumber Data Utama
| File | Isi |
|------|-----|
| `LKPS Prodi RKS 2025 (2).docx` | Data DTPR, mahasiswa, lulusan, penelitian, PkM, rekognisi — sumber data utama untuk narasi Pelaksanaan |
| `STANDAR MUTU LED RKS.xlsx` | Standar mutu dan indikator resmi per kriteria (sheet tunggal "Indikator Mutu LED") — acuan Penetapan |
| `LED Prodi RKS 2025.docx` | Draft LED sebelumnya — referensi narasi dan data evaluasi |
| `laporan AMI RKS 2025.pdf` | Hasil AMI: 44 temuan (37 OB + 7 KTS), nilai 87,00 |
| `laporan_rekap_desk_evaluation_detail_18042026.xls` | Data desk evaluation per indikator, 18 April 2026 |
| `panduan-led-lkps-rks-laminfokom-2.1.md` | Panduan lengkap penyusunan LED & LKPS untuk instrumen ini |

## Cara Membuat Dokumen LED Baru

### Pola standar per dokumen PPEPP

**Penetapan (CX.1):** kebijakan (numbered list peraturan) → penjelasan sub-aspek → tabel standar mutu (kolom: No | Pernyataan Standar | Indikator | Target | Standar Mutu)

**Pelaksanaan (CX.2):** narasi per sub-aspek dengan data aktual LKPS → tabel realisasi (kolom: No | Indikator | Target | Realisasi | Keterangan)

**Evaluasi (CX.3):** konteks AMI 2025 → narasi hasil per sub-aspek → tabel evaluasi (kolom: No | Indikator | Target | Realisasi | Hasil Evaluasi | Kategori AMI)

**Pengendalian (CX.4):** tindak lanjut KTS/Observasi (akar masalah + langkah-langkah) → tabel pengendalian (kolom: No | Indikator | Hasil Evaluasi | Kategori | Akar Masalah | Tindak Lanjut | PJ | Status TL)

**Peningkatan (CX.5):** target baru + strategi per sub-aspek → tabel peningkatan (kolom: No | Indikator | Target Lama | Hasil Evaluasi | Target Baru | Strategi Peningkatan)

### Membuat file gabungan

```python
import re

def bump_headings(content, n=1):
    lines = content.split('\n')
    result = []
    for line in lines:
        if re.match(r'^#{1,6}\s', line):
            result.append('#' * n + line)
        else:
            result.append(line)
    return '\n'.join(result)

files = ['LED-CX.1-*.md', 'LED-CX.2-*.md', ...]
parts = [bump_headings(open(f).read(), 1) for f in files]
combined = '\n\n---\n\n'.join(parts)
open('LED-CX-*.md', 'w').write(combined)
```

### Membaca STANDAR MUTU LED RKS.xlsx (tanpa openpyxl)

```python
import zipfile, xml.etree.ElementTree as ET

with zipfile.ZipFile("STANDAR MUTU LED RKS.xlsx") as z:
    ns = {'ns': 'http://schemas.openxmlformats.org/spreadsheetml/2006/main'}
    shared = [si.text or ''.join(t.text or '' for t in si.iter())
              for si in ET.parse(z.open('xl/sharedStrings.xml')).findall('.//ns:si', ns)]
    rows = ET.parse(z.open('xl/worksheets/sheet1.xml')).findall('.//ns:row', ns)
    for row in rows:
        cells = []
        for c in row.findall('ns:c', ns):
            v = c.find('ns:v', ns)
            cells.append(shared[int(v.text)] if v is not None and c.get('t') == 's'
                         else (v.text if v is not None else ''))
```

### Membaca LKPS .docx

```python
from docx import Document
doc = Document("LKPS Prodi RKS 2025 (2).docx")
# Tabel utama yang sering dipakai:
# doc.tables[4]  → DTPR (beban kerja, nama dosen)
# doc.tables[7]  → SPMB (daya tampung, pendaftar, diterima)
# doc.tables[8]  → Asal mahasiswa per provinsi
# doc.tables[9]  → Mahasiswa aktif & lulus per tahun
# doc.tables[17] → Bentuk pembelajaran (CBL/PBL, pertukaran)
# doc.tables[18] → Rekognisi lulusan (CPNS BSSN)
# doc.tables[20] → Penelitian DTPR
# doc.tables[21] → Pengembangan DTPR
# doc.tables[23] → Publikasi DTPR
```

## Data Kunci PS RKS (Tidak Perlu Dicari Ulang)

- **15 DTPR** | S3: 4 (Amiruddin, Magfirawaty, Prasetyo, Susila Windarta) | Lektor Kepala: 1 (Amiruddin) — **ini KTS di C.2**
- **Mahasiswa aktif TS**: 198 | Rasio DTPR:mhs = 1:13
- **SPMB TS**: 3.183 pendaftar, 42 diterima, rasio 1:75 | 20 provinsi asal
- **Lulusan**: TS-2=55, TS-1=41, TS=54 | 100% CPNS BSSN Gol. III/a | masa tunggu 0 tahun
- **AMI 2025**: 44 temuan (37 OB + 7 KTS) | nilai 87,00 | perkiraan akreditasi 82,00
- **Penelitian**: 63 judul (6 tema) | Scopus: Magfirawaty (chaos RNG), Dimas (OODA, PROCTOR)
- **Lab**: 6 (Lab Siber 1&2, Lab SOC, Lab Forensic Digital, Lab Smart City, Lab Data Center)
- **Dana pendidikan TS**: Rp 3.397,1 juta dari total Rp 3.793,2 juta (89,6% dari tridharma)
- **VMTS**: Kepdir No. 06.2 Tahun 2022 | dievaluasi ≤2027
- **SKKNI rujukan**: 391/2020 (SOC), 23/2022 (Uji Keamanan), 24/2022 (Audit), 4/2023 (Kriptografi)

## Placeholder yang Belum Terisi

- `[No SK]` — Nomor SK Standar Mutu 2025 (konfirmasi ke Pus Jamut)
- `[Link]` — Tautan Google Drive bukti dukung
- Data dalam verifikasi di C.2: IPK/IPS lulusan, nilai sikap, nilai kesemaptaan, sertifikat bahasa Inggris, jumlah lokasi SKD, prestasi mahasiswa

## Konvensi Penulisan

- Semua heading file individual dimulai dari `#` (H1); saat digabung menjadi `##` (H2) via `bump_headings`
- Placeholder data yang belum tersedia: *[data dikonfirmasi ke Unit X]*
- Kategori temuan AMI: **KTS** (Ketidaksesuaian) atau **Observasi (OB)**
- Status tindak lanjut: *Selesai* / *Dalam proses* / *Belum dimulai*
- Footer setiap dokumen: `*Dokumen: LED-CX.Y-Nama.md — 21 April 2026*` + baris sumber
