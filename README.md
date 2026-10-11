
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tabel Keuangan</title>

<style>
*,
*:before,
*:after {
    box-sizing: border-box;
}

body {
    margin: 0;
    padding: 12px;
    font-family: Arial, sans-serif;
    background: #eef4fa;
    color: #263238;
}

h2 {
    text-align: center;
    color: #1769aa;
    font-size: 22px;
    margin: 12px 0 18px;
}

.toolbar {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    margin-bottom: 12px;
}

.btn {
    padding: 11px 15px;
    border: none;
    border-radius: 7px;
    color: white;
    font-size: 14px;
    font-weight: bold;
    cursor: pointer;
}

.btn-save { background: #087f8c; }
.btn-add { background: #218838; }
.btn-delete { background: #dc3545; }

.btn:active {
    opacity: 0.75;
}

.status {
    font-size: 13px;
    color: #187447;
    margin: 8px 0 12px;
}

.table-container {
    width: 100%;
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 8px #00000012;
}

table {
    width: 100%;
    min-width: 720px;
    border-collapse: collapse;
    background: white;
}

th, td {
    border: 1px solid #263238;
    padding: 6px;
    vertical-align: middle;
}

th {
    background: #1769aa;
    color: white;
    text-align: center;
    font-size: 13px;
    padding: 12px 7px;
}

tbody tr:nth-child(even) {
    background: #f0f7ff;
}

td input {
    display: block;
    width: 100%;
    min-width: 0;
    padding: 9px 5px;
    border: 1px solid #b8c9d8;
    border-radius: 4px;
    background: white;
    color: #222;
    font-size: 14px;
}

td input:focus {
    outline: 2px solid #4da3ff;
    border-color: #1769aa;
}

input[type="number"] {
    text-align: right;
}

.saldo-cell {
    background: #e1f5e7;
    font-weight: bold;
    text-align: right;
    min-width: 100px;
    white-space: nowrap;
}

.text-center {
    text-align: center;
}

.total-row td {
    background: #d4e7f7 !important;
    font-weight: bold;
    padding: 12px 7px;
    border-top: 3px solid #263238;
}

.total-label {
    text-align: right;
}

tfoot td {
    white-space: nowrap;
}

.note {
    font-size: 12px;
    color: #526574;
    margin-top: 12px;
    line-height: 1.6;
}

@media (max-width: 480px) {
    body {
        padding: 8px;
    }

    h2 {
        font-size: 19px;
    }

    .btn {
        padding: 10px 12px;
    }
}
</style>
</head>

<body>

<h2>TABEL KEUANGAN</h2>

<div class="toolbar">
    <button class="btn btn-save" onclick="simpanData()">
        💾 Simpan
    </button>

    <button class="btn btn-add" onclick="tambahBaris()">
        + Tambah Baris
    </button>

    <button class="btn btn-delete" onclick="hapusSemua()">
        Hapus Semua
    </button>
</div>

<div class="status" id="status">
    Data siap diisi.
</div>

<div class="table-container">
<table>
    <thead>
        <tr>
            <th style="min-width:145px">Tanggal</th>
            <th style="min-width:160px">Nama / Keterangan</th>
            <th style="min-width:120px">Debet</th>
            <th style="min-width:120px">Kredit</th>
            <th style="min-width:125px">Saldo</th>
            <th style="min-width:65px">Aksi</th>
        </tr>
    </thead>

    <tbody id="isi-tabel">
    </tbody>

    <tfoot>
        <tr class="total-row">
            <td colspan="2" class="total-label">
                TOTAL:
            </td>
            <td id="total-debet" class="text-center">0</td>
            <td id="total-kredit" class="text-center">0</td>
            <td id="total-saldo" class="text-center">0</td>
            <td></td>
        </tr>
    </tfoot>
</table>
</div>

<div class="note">
    • Isi tanggal, nama, debet dan kredit secara manual.<br>
    • Saldo dihitung otomatis: saldo sebelumnya + debet − kredit.<br>
    • Total saldo menunjukkan saldo akhir transaksi, bukan penjumlahan semua saldo berjalan.<br>
    • Geser tabel ke kiri atau kanan untuk melihat semua kolom.
</div>

<script>
const KUNCI_DATA = "tabelKeuanganAndroid_v1";

const tabel = document.getElementById("isi-tabel");
const statusEl = document.getElementById("status");

function tanggalHariIni() {
    const sekarang = new Date();
    const tahun = sekarang.getFullYear();
    const bulan = String(sekarang.getMonth() + 1).padStart(2, "0");
    const hari = String(sekarang.getDate()).padStart(2, "0");

    return `${tahun}-${bulan}-${hari}`;
}

function angka(nilai) {
    const hasil = Number(nilai);
    return Number.isFinite(hasil) ? hasil : 0;
}

function formatAngka(nilai) {
    return nilai.toLocaleString("id-ID", {
        maximumFractionDigits: 2
    });
}

function buatInput(tipe, nilai, placeholder) {
    const input = document.createElement("input");

    input.type = tipe;
    input.value = nilai ?? "";
    input.placeholder = placeholder || "";

    if (tipe === "number") {
        input.step = "any";
        input.inputMode = "decimal";
    }

    input.addEventListener("input", function() {
        hitungSaldo();
        simpanOtomatis();
    });

    return input;
}

function tambahBaris(data = {}) {
    const tr = document.createElement("tr");

    const tdTanggal = document.createElement("td");
    const tdNama = document.createElement("td");
    const tdDebet = document.createElement("td");
    const tdKredit = document.createElement("td");
    const tdSaldo = document.createElement("td");
    const tdAksi = document.createElement("td");

    const inputTanggal = buatInput(
        "date",
        data.tanggal ?? tanggalHariIni()
    );

    const inputNama = buatInput(
        "text",
        data.nama ?? "",
        "Nama transaksi"
    );

    const inputDebet = buatInput(
        "number",
        data.debet ?? "",
        "0"
    );

    const inputKredit = buatInput(
        "number",
        data.kredit ?? "",
        "0"
    );

    tdTanggal.appendChild(inputTanggal);
    tdNama.appendChild(inputNama);
    tdDebet.appendChild(inputDebet);
    tdKredit.appendChild(inputKredit);

    tdSaldo.className = "saldo-cell";
    tdSaldo.textContent = "0";

    const btnHapus = document.createElement("button");
    btnHapus.type = "button";
    btnHapus.className = "btn btn-delete";
    btnHapus.textContent = "X";
    btnHapus.title = "Hapus baris";

    btnHapus.addEventListener("click", function() {
        if (confirm("Hapus transaksi ini?")) {
            tr.remove();

            if (tabel.rows.length === 0) {
                tambahBaris();
            }

            hitungSaldo();
            simpanOtomatis();
        }
    });

    tdAksi.className = "text-center";
    tdAksi.appendChild(btnHapus);

    tr.append(
        tdTanggal,
        tdNama,
        tdDebet,
        tdKredit,
        tdSaldo,
        tdAksi
    );

    tabel.appendChild(tr);

    hitungSaldo();
}

function hitungSaldo() {
    let totalDebet = 0;
    let totalKredit = 0;
    let saldoBerjalan = 0;

    Array.from(tabel.rows).forEach(function(tr) {
        const input = tr.querySelectorAll("input");

        const debet = angka(input[2].value);
        const kredit = angka(input[3].value);

        totalDebet += debet;
        totalKredit += kredit;

        saldoBerjalan += debet - kredit;

        tr.cells[4].textContent = formatAngka(saldoBerjalan);
    });

    document.getElementById("total-debet").textContent =
        formatAngka(totalDebet);

    document.getElementById("total-kredit").textContent =
        formatAngka(totalKredit);

    document.getElementById("total-saldo").textContent =
        formatAngka(saldoBerjalan);
}

function ambilData() {
    return Array.from(tabel.rows).map(function(tr) {
        const input = tr.querySelectorAll("input");

        return {
            tanggal: input[0].value,
            nama: input[1].value,
            debet: input[2].value,
            kredit: input[3].value
        };
    });
}

function simpanOtomatis() {
    try {
        localStorage.setItem(
            KUNCI_DATA,
            JSON.stringify(ambilData())
        );

        statusEl.textContent = "Perubahan tersimpan otomatis.";
    } catch (error) {
        statusEl.textContent =
            "Gagal menyimpan. Periksa penyimpanan browser.";
    }
}

function simpanData() {
    try {
        localStorage.setItem(
            KUNCI_DATA,
            JSON.stringify(ambilData())
        );

        statusEl.textContent = "Data berhasil disimpan di browser ini.";
        alert("Data berhasil disimpan!");
    } catch (error) {
        alert("Gagal menyimpan data. Periksa penyimpanan browser.");
    }
}

function muatData() {
    try {
        const tersimpan = localStorage.getItem(KUNCI_DATA);

        if (tersimpan) {
            const data = JSON.parse(tersimpan);

            if (Array.isArray(data) && data.length > 0) {
                data.forEach(function(item) {
                    tambahBaris(item);
                });

                hitungSaldo();
                statusEl.textContent = "Data tersimpan berhasil dimuat.";
                return;
            }
        }
    } catch (error) {
        statusEl.textContent =
            "Data lama tidak dapat dibaca. Silakan periksa browser.";
    }

    tambahBaris();
}

function hapusSemua() {
    if (!confirm("Yakin ingin menghapus SEMUA transaksi?")) {
        return;
    }

    try {
        localStorage.removeItem(KUNCI_DATA);
        tabel.innerHTML = "";

        tambahBaris();
        hitungSaldo();

        statusEl.textContent = "Semua data telah dihapus.";
    } catch (error) {
        alert("Penyimpanan browser tidak dapat diakses.");
    }
}

muatData();
</script>

</body>
</html>
