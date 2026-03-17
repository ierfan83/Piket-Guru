<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Piket Guru - Sistem Kendali Sekolah</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <!-- Library html2pdf untuk konversi PDF langsung -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; transition: background-color 0.3s; }
        .glass-card { background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(10px); }
        .tab-content { display: none; }
        .tab-content.active { display: block; }

        /* Gaya Cetak / Print Styles */
        @media print {
            body * { visibility: hidden; }
            #printArea, #printArea * { visibility: visible; }
            #printArea {
                position: absolute;
                left: 0;
                top: 0;
                width: 100%;
                padding: 40px;
                background: white;
            }
            .no-print { display: none !important; }
        }
    </style>
</head>
<body class="bg-gray-50 text-gray-800">

    <!-- Navigasi Utama -->
    <nav class="bg-indigo-700 text-white shadow-lg sticky top-0 z-50 no-print">
        <div class="container mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i class="fas fa-shield-halved text-2xl"></i>
                <span class="font-bold text-xl tracking-tight">SmartPiket</span>
            </div>
            <div id="navLinks" class="hidden md:flex space-x-6 items-center">
                <button onclick="showTab('dashboard')" class="hover:text-indigo-200 transition font-medium">Dashboard</button>
                <button onclick="showTab('surat')" class="hover:text-indigo-200 transition font-medium">Surat Izin</button>
                <button onclick="showTab('inval')" class="hover:text-indigo-200 transition font-medium">Koordinasi Inval</button>
                <button onclick="showTab('absensi')" class="hover:text-indigo-200 transition font-medium">Absensi</button>
                <button onclick="logout()" id="logoutBtn" class="bg-red-500 px-4 py-1 rounded hover:bg-red-600 hidden text-sm">Logout</button>
                <button onclick="showLogin()" id="loginBtn" class="bg-indigo-500 px-4 py-1 rounded hover:bg-indigo-600 text-sm">Login Admin</button>
            </div>
        </div>
    </nav>

    <main class="container mx-auto px-4 py-8 no-print">

        <!-- DASHBOARD -->
        <div id="dashboard" class="tab-content active space-y-6">
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <!-- Info Waktu -->
                <div class="glass-card p-6 rounded-2xl shadow-sm border border-gray-100">
                    <h2 class="text-xs font-bold text-gray-400 uppercase tracking-wider">Status Hari Ini</h2>
                    <p id="currentDate" class="text-xl font-bold mt-1 text-indigo-900"></p>
                    <div class="mt-4 flex items-center text-indigo-600 bg-indigo-50 p-2 rounded-lg w-max">
                        <i class="fas fa-clock mr-2"></i>
                        <span id="currentTime" class="font-mono font-bold">--:--:--</span>
                    </div>
                </div>

                <!-- Bel Digital -->
                <div class="glass-card p-6 rounded-2xl shadow-sm border border-gray-100">
                    <h2 class="text-xs font-bold text-gray-400 uppercase tracking-wider mb-4">Bel Sekolah</h2>
                    <div class="grid grid-cols-2 gap-2">
                        <button onclick="playBell('masuk')" class="bg-blue-600 text-white py-2 rounded-lg text-sm hover:bg-blue-700">Masuk</button>
                        <button onclick="playBell('istirahat')" class="bg-yellow-500 text-white py-2 rounded-lg text-sm hover:bg-yellow-600">Istirahat</button>
                        <button onclick="playBell('pulang')" class="bg-red-600 text-white py-2 rounded-lg text-sm hover:bg-red-700">Pulang</button>
                        <button onclick="playBell('darurat')" class="bg-black text-white py-2 rounded-lg text-sm animate-pulse">Darurat</button>
                    </div>
                </div>

                <!-- Guru Piket Aktif -->
                <div class="glass-card p-6 rounded-2xl shadow-sm border border-gray-100">
                    <h2 class="text-xs font-bold text-gray-400 uppercase tracking-wider mb-3">Piket Aktif</h2>
                    <ul id="listPiketToday" class="space-y-2"></ul>
                </div>
            </div>

            <!-- Pintasan Surat -->
            <div class="bg-gradient-to-r from-indigo-600 to-indigo-800 p-8 rounded-2xl shadow-lg text-white flex flex-col md:flex-row items-center justify-between">
                <div class="mb-4 md:mb-0">
                    <h3 class="font-bold text-2xl">Layanan Surat Izin Digital</h3>
                    <p class="opacity-80">Generate surat izin masuk/keluar dengan Nama & NIP guru piket otomatis.</p>
                </div>
                <button onclick="showTab('surat')" class="bg-white text-indigo-700 font-bold px-8 py-3 rounded-xl hover:bg-gray-100 transition shadow-md">
                    Buat Surat Sekarang
                </button>
            </div>
        </div>

        <!-- MENU SURAT IZIN -->
        <div id="surat" class="tab-content space-y-6">
            <h2 class="text-2xl font-bold text-indigo-900">Layanan Surat Izin Siswa</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <!-- Form Input -->
                <div class="bg-white p-6 rounded-2xl shadow-md border border-gray-100">
                    <h3 class="font-bold mb-6 text-indigo-700 flex items-center">
                        <i class="fas fa-file-signature mr-2"></i> Formulir Surat Izin
                    </h3>
                    <div class="space-y-4">
                        <div>
                            <label class="text-xs font-bold text-gray-500 block mb-1">Pilih Guru Piket</label>
                            <select id="izinGuruPiket" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none transition"></select>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="text-xs font-bold text-gray-500 block mb-1">Data Siswa</label>
                                <input type="text" id="izinNama" placeholder="Nama Siswa" class="w-full p-3 border border-gray-200 rounded-xl mb-3 focus:ring-2 ring-indigo-200 outline-none">
                            </div>
                            <div>
                                <label class="text-xs font-bold text-gray-500 block mb-1">Kelas</label>
                                <input type="text" id="izinKelas" placeholder="Contoh: X-IPA 1" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none">
                            </div>
                        </div>
                        <div class="grid grid-cols-2 gap-3">
                            <div>
                                <label class="text-xs font-bold text-gray-500 block mb-1">Tipe Izin</label>
                                <select id="izinTipe" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none">
                                    <option value="Keluar">Izin Keluar Lingkungan Sekolah</option>
                                    <option value="Masuk">Izin Masuk Kelas (Terlambat)</option>
                                    <option value="Kembali">Izin Kembali ke Kelas</option>
                                </select>
                            </div>
                            <div>
                                <label class="text-xs font-bold text-gray-500 block mb-1">Tanggal Izin</label>
                                <input type="date" id="izinTanggal" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none">
                            </div>
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500 block mb-1">Alasan</label>
                            <textarea id="izinAlasan" placeholder="Tulis alasan izin..." class="w-full p-3 border border-gray-200 rounded-xl h-24 focus:ring-2 ring-indigo-200 outline-none"></textarea>
                        </div>
                        <button onclick="generateSurat()" class="w-full bg-indigo-600 text-white py-4 rounded-xl font-bold hover:bg-indigo-700 transition shadow-md">
                            Generate & Preview Surat
                        </button>
                    </div>
                </div>

                <!-- Preview Box -->
                <div class="flex flex-col">
                    <div id="previewSurat" class="bg-white p-10 shadow-xl border border-gray-200 rounded-xl min-h-[400px] hidden text-sm leading-relaxed overflow-hidden">
                        <!-- Konten preview akan muncul di sini -->
                    </div>
                    <div id="previewPlaceholder" class="bg-gray-100 p-10 rounded-2xl border-2 border-dashed border-gray-300 flex flex-col items-center justify-center flex-grow text-gray-400">
                        <i class="fas fa-file-pdf text-6xl mb-4"></i>
                        <p class="text-center">Silahkan lengkapi formulir di samping untuk melihat preview surat yang siap dicetak.</p>
                    </div>
                    <button id="btnPrintSurat" onclick="downloadPDF()" class="mt-6 bg-green-600 text-white py-4 rounded-xl font-bold hover:bg-green-700 transition shadow-lg hidden flex items-center justify-center">
                        <i class="fas fa-download mr-2"></i> Download PDF Langsung
                    </button>
                </div>
            </div>
        </div>

        <!-- KOORDINASI INVAL -->
        <div id="inval" class="tab-content space-y-6">
            <h2 class="text-2xl font-bold text-indigo-900">Pencarian Guru Inval</h2>
            <div class="bg-white p-8 rounded-2xl shadow-md border border-gray-100">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-6">
                    <div>
                        <label class="block text-sm font-bold text-gray-600 mb-2">Kelas Kosong</label>
                        <select class="w-full border p-3 rounded-xl bg-gray-50 outline-none focus:ring-2 ring-indigo-200">
                            <option>X-IPA 1</option><option>XI-IPA 2</option><option>XII-IPS 1</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm font-bold text-gray-600 mb-2">Jam Pelajaran</label>
                        <input type="number" value="1" class="w-full border p-3 rounded-xl bg-gray-50 outline-none focus:ring-2 ring-indigo-200">
                    </div>
                    <div class="flex items-end">
                        <button onclick="cariInval()" class="w-full bg-indigo-600 text-white py-3 rounded-xl font-bold hover:bg-indigo-700 shadow-md transition">Cari Guru Tersedia</button>
                    </div>
                </div>
                <div id="hasilInval" class="hidden border-t pt-6">
                    <p class="text-sm text-gray-500 mb-4 font-medium italic">Saran Guru Inval (Berdasarkan jadwal kosong):</p>
                    <div class="p-4 bg-indigo-50 rounded-xl border border-indigo-100 flex justify-between items-center">
                        <div>
                            <span class="font-bold text-indigo-900 block">Rina Marlina, S.Pd</span>
                            <span class="text-xs text-indigo-600">Guru B. Inggris - Jam Kosong</span>
                        </div>
                        <button class="bg-indigo-600 text-white px-5 py-2 rounded-lg text-sm hover:bg-indigo-700 transition">Hubungi</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- ABSENSI & LAPORAN -->
        <div id="absensi" class="tab-content space-y-6">
            <h2 class="text-2xl font-bold text-indigo-900">Manajemen Absensi & Laporan</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <div class="bg-white p-8 rounded-2xl shadow-md border border-gray-100">
                    <h3 class="font-bold mb-6 flex items-center text-red-600">
                        <i class="fas fa-user-minus mr-2"></i> Lapor Ketidakhadiran Siswa
                    </h3>
                    <form onsubmit="event.preventDefault(); showToast('Laporan absensi berhasil dikirim'); this.reset();" class="space-y-4">
                        <input type="text" placeholder="Nama Siswa" class="w-full p-3 border border-gray-200 rounded-xl" required>
                        <select class="w-full p-3 border border-gray-200 rounded-xl bg-white">
                            <option>Sakit</option><option>Izin</option><option>Alpa</option><option>Tanpa Keterangan</option>
                        </select>
                        <button type="submit" class="w-full bg-red-500 text-white py-3 rounded-xl font-bold hover:bg-red-600 transition shadow-md">Kirim Laporan</button>
                    </form>
                </div>
                <div class="bg-white p-8 rounded-2xl shadow-md border border-gray-100">
                    <h3 class="font-bold mb-6 flex items-center text-indigo-600">
                        <i class="fas fa-book-open mr-2"></i> Jurnal Harian Piket
                    </h3>
                    <textarea placeholder="Tuliskan kejadian penting atau catatan piket hari ini..." class="w-full p-3 border border-gray-200 rounded-xl h-32"></textarea>
                    <button onclick="showToast('Jurnal piket berhasil disimpan')" class="w-full bg-indigo-600 text-white py-3 rounded-xl font-bold mt-4 hover:bg-indigo-700 transition shadow-md">Simpan Jurnal</button>
                </div>
            </div>
        </div>

        <!-- PANEL ADMIN (DATA GURU PIKET) -->
        <div id="admin" class="tab-content space-y-6">
            <div class="flex items-center justify-between">
                <h2 class="text-2xl font-bold text-indigo-900">Panel Administrasi Sekolah</h2>
                <div class="bg-indigo-100 text-indigo-700 px-4 py-2 rounded-lg text-sm font-bold">Mode Admin Aktif</div>
            </div>
            
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                <!-- PENGATURAN IDENTITAS SEKOLAH -->
                <div class="bg-white p-8 rounded-2xl shadow-md border border-indigo-100">
                    <h3 class="font-bold mb-6 text-indigo-700 flex items-center">
                        <i class="fas fa-school mr-2"></i> Identitas Sekolah (PDF)
                    </h3>
                    <div class="space-y-4">
                        <div>
                            <label class="text-xs font-bold text-gray-500 uppercase tracking-widest block mb-1">Instansi Utama (Kop 1)</label>
                            <input type="text" id="setInstansi1" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none text-sm">
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500 uppercase tracking-widest block mb-1">Instansi Kedua (Kop 2)</label>
                            <input type="text" id="setInstansi2" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none text-sm">
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500 uppercase tracking-widest block mb-1">Nama Sekolah</label>
                            <input type="text" id="setSekolah" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none text-sm font-bold">
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500 uppercase tracking-widest block mb-1">Alamat Sekolah</label>
                            <textarea id="setAlamat" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none text-sm h-20"></textarea>
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500 uppercase tracking-widest block mb-1">Nama Kota (Untuk TTD)</label>
                            <input type="text" id="setKota" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none text-sm">
                        </div>
                        <button onclick="updateSchoolData()" class="w-full bg-indigo-600 text-white py-3 rounded-xl font-bold hover:bg-indigo-700 transition shadow-md">
                            Perbarui Data Sekolah
                        </button>
                    </div>
                </div>

                <!-- FORM TAMBAH GURU -->
                <div class="bg-white p-8 rounded-2xl shadow-md border border-gray-100">
                    <h3 class="font-bold mb-6 text-indigo-700 flex items-center">
                        <i class="fas fa-user-plus mr-2"></i> Tambah Guru Piket
                    </h3>
                    <form id="formTambahGuru" class="space-y-4">
                        <div>
                            <label class="text-xs font-bold text-gray-500 uppercase tracking-widest block mb-1">Nama Lengkap & Gelar</label>
                            <input type="text" id="inputNama" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none" placeholder="Contoh: Dr. Jaka Kelana, M.Pd" required>
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500 uppercase tracking-widest block mb-1">NIP / Nomor Induk</label>
                            <input type="text" id="inputNIP" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none" placeholder="19XXXXXXXX XXXXXX X XXX" required>
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500 uppercase tracking-widest block mb-1">Jenis Kelamin</label>
                            <div class="flex space-x-4 mt-2">
                                <label class="flex items-center cursor-pointer">
                                    <input type="radio" name="jk" value="Laki-laki" checked class="mr-2 accent-indigo-600"> Laki-laki
                                </label>
                                <label class="flex items-center cursor-pointer">
                                    <input type="radio" name="jk" value="Perempuan" class="mr-2 accent-indigo-600"> Perempuan
                                </label>
                            </div>
                        </div>
                        <div>
                            <label class="text-xs font-bold text-gray-500 uppercase tracking-widest block mb-1">Hari Bertugas</label>
                            <select id="inputHari" class="w-full p-3 border border-gray-200 rounded-xl focus:ring-2 ring-indigo-200 outline-none">
                                <option>Senin</option><option>Selasa</option><option>Rabu</option><option>Kamis</option><option>Jumat</option><option>Sabtu</option>
                            </select>
                        </div>
                        <button type="submit" class="w-full bg-indigo-600 text-white py-4 rounded-xl font-bold hover:bg-indigo-700 transition shadow-lg mt-4">
                            Simpan Data Guru
                        </button>
                    </form>
                </div>

                <!-- DAFTAR GURU -->
                <div class="bg-white p-8 rounded-2xl shadow-md border border-gray-100 overflow-hidden">
                    <h3 class="font-bold mb-6 text-gray-700">Database Guru Piket</h3>
                    <div class="overflow-y-auto max-h-[500px]">
                        <table class="min-w-full text-sm">
                            <thead class="bg-gray-50 text-gray-500 font-bold uppercase text-[10px] tracking-widest">
                                <tr>
                                    <th class="px-4 py-4 text-left border-b">Detail Guru</th>
                                    <th class="px-4 py-4 text-center border-b">Hapus</th>
                                </tr>
                            </thead>
                            <tbody id="tabelGuruAdmin" class="text-gray-700"></tbody>
                        </table>
                    </div>
                    <div id="noDataAdmin" class="hidden py-10 text-center text-gray-400">
                        <i class="fas fa-users-slash text-4xl mb-2"></i>
                        <p>Belum ada data guru piket.</p>
                    </div>
                </div>
            </div>
        </div>

    </main>

    <!-- AREA CETAK SURAT (HIDDEN FROM UI, USED BY PDF GENERATOR) -->
    <div id="printArea" class="hidden" style="background: white; padding: 40px; color: black; font-family: 'Times New Roman', Times, serif;">
        <div style="text-align: center; border-bottom: 3px double #000; padding-bottom: 15px; margin-bottom: 30px;">
            <h1 id="printKopInstansi1" style="margin: 0; font-size: 16px; font-weight: bold; text-transform: uppercase;">PEMERINTAH PROVINSI BENGKULU</h1>
            <h1 id="printKopInstansi2" style="margin: 0; font-size: 16px; font-weight: bold; text-transform: uppercase;">DINAS PENDIDIKAN DAN KEBUDAYAAN</h1>
            <h2 id="printKopSekolah" style="margin: 0; font-size: 20px; font-weight: bold; text-transform: uppercase;">SMA NEGERI 10 MUKOMUKO</h2>
            <p id="printKopAlamat" style="margin: 5px 0 0; font-size: 11px;">Jl. Teluk Beringin, Penarik, Kec. Penarik, Kabupaten Mukomuko, Provinsi Bengkulu.</p>
        </div>
        <div id="printContent" style="line-height: 1.6; font-size: 14px;"></div>
        <div style="margin-top: 60px; display: flex; justify-content: space-between;">
            <div style="text-align: center; width: 200px;">
                <p>Siswa Yang Bersangkutan,</p>
                <br><br><br>
                <p id="printNamaSiswa" style="font-weight: bold; text-decoration: underline;">( ................................ )</p>
            </div>
            <div style="text-align: center; width: 250px;">
                <p id="printLabelTtdKota"></p>
                <p>Guru Piket Penanggung Jawab,</p>
                <br><br><br>
                <p id="printTtdGuru" style="font-weight: bold; text-decoration: underline;">( ................................ )</p>
                <p id="printNipGuru" style="font-size: 12px; margin-top: 2px;"></p>
            </div>
        </div>
        <div style="margin-top: 40px; text-align: center; font-size: 9px; color: #555; border-top: 1px solid #ccc; padding-top: 5px;">
            Dokumen sah dicetak secara sistem melalui SmartPiket pada <span id="printWaktu"></span>
        </div>
    </div>

    <!-- MODAL LOGIN -->
    <div id="loginModal" class="fixed inset-0 bg-black/60 backdrop-blur-sm hidden flex items-center justify-center p-4 z-[100] no-print">
        <div class="bg-white w-full max-w-md rounded-3xl p-8 shadow-2xl transform transition-all">
            <div class="text-center mb-8">
                <div class="w-16 h-16 bg-indigo-100 text-indigo-600 rounded-full flex items-center justify-center mx-auto mb-4">
                    <i class="fas fa-user-shield text-3xl"></i>
                </div>
                <h2 class="text-2xl font-bold text-gray-800">Login Administrator</h2>
                <p class="text-gray-500 text-sm">Gunakan akun admin untuk mengelola data</p>
            </div>
            <form id="loginForm" class="space-y-4">
                <div class="relative">
                    <i class="fas fa-user absolute left-4 top-4 text-gray-400"></i>
                    <input type="text" id="username" class="w-full p-4 pl-12 border border-gray-200 rounded-2xl focus:ring-2 ring-indigo-200 outline-none" placeholder="Username">
                </div>
                <div class="relative">
                    <i class="fas fa-lock absolute left-4 top-4 text-gray-400"></i>
                    <input type="password" id="password" class="w-full p-4 pl-12 border border-gray-200 rounded-2xl focus:ring-2 ring-indigo-200 outline-none" placeholder="Password">
                </div>
                <button type="submit" class="w-full bg-indigo-600 text-white py-4 rounded-2xl font-bold hover:bg-indigo-700 transition shadow-lg mt-4">Login Sekarang</button>
                <button type="button" onclick="closeLogin()" class="w-full text-gray-400 mt-2 text-sm text-center font-medium hover:text-gray-600 transition">Kembali ke Beranda</button>
            </form>
        </div>
    </div>

    <!-- Toast -->
    <div id="toast" class="fixed bottom-6 right-6 bg-gray-900 text-white px-8 py-4 rounded-2xl shadow-2xl opacity-0 transition-all pointer-events-none z-[1000] flex items-center">
        <i class="fas fa-info-circle mr-3 text-indigo-400"></i>
        <span id="toastMsg"></span>
    </div>

    <script>
        // Data Default
        const defaultSchool = {
            instansi1: "PEMERINTAH PROVINSI BENGKULU",
            instansi2: "DINAS PENDIDIKAN DAN KEBUDAYAAN",
            sekolah: "SMA NEGERI 10 MUKOMUKO",
            alamat: "Jl. Teluk Beringin, Penarik, Kec. Penarik, Kabupaten Mukomuko, Provinsi Bengkulu.",
            kota: "Mukomuko"
        };

        // Inisialisasi State
        let isAdmin = false;
        let dbGuru = JSON.parse(localStorage.getItem('dbGuru')) || [];
        let schoolData = JSON.parse(localStorage.getItem('schoolData')) || defaultSchool;

        window.onload = () => {
            initAdminFields();
            renderGuruList();
            renderPiketToday();
            populateGuruPiketSelect();
            updateTime();
            document.getElementById('izinTanggal').valueAsDate = new Date();
            if (localStorage.getItem('smartPiketAuth') === 'true') {
                isAdmin = true;
                document.getElementById('loginBtn').classList.add('hidden');
                document.getElementById('logoutBtn').classList.remove('hidden');
            }
        };

        function initAdminFields() {
            document.getElementById('setInstansi1').value = schoolData.instansi1;
            document.getElementById('setInstansi2').value = schoolData.instansi2;
            document.getElementById('setSekolah').value = schoolData.sekolah;
            document.getElementById('setAlamat').value = schoolData.alamat;
            document.getElementById('setKota').value = schoolData.kota;
        }

        function updateSchoolData() {
            schoolData = {
                instansi1: document.getElementById('setInstansi1').value,
                instansi2: document.getElementById('setInstansi2').value,
                sekolah: document.getElementById('setSekolah').value,
                alamat: document.getElementById('setAlamat').value,
                kota: document.getElementById('setKota').value
            };
            localStorage.setItem('schoolData', JSON.stringify(schoolData));
            showToast("Identitas sekolah berhasil diperbarui!");
        }

        // Real-time Clock
        function updateTime() {
            const now = new Date();
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
            document.getElementById('currentDate').innerText = now.toLocaleDateString('id-ID', options);
            document.getElementById('currentTime').innerText = now.toLocaleTimeString('id-ID', { hour12: false });
        }
        setInterval(updateTime, 1000);

        // Tab Navigation
        function showTab(tabId) {
            if (tabId === 'admin' && !isAdmin) {
                showToast("Akses Dibatasi! Silahkan Login sebagai Admin.");
                return;
            }
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.getElementById(tabId).classList.add('active');
            window.scrollTo({ top: 0, behavior: 'smooth' });
        }

        // --- MANAJEMEN ADMIN (CRUD GURU) ---
        document.getElementById('formTambahGuru').onsubmit = function(e) {
            e.preventDefault();
            const nama = document.getElementById('inputNama').value;
            const nip = document.getElementById('inputNIP').value;
            const jk = document.querySelector('input[name="jk"]:checked').value;
            const hari = document.getElementById('inputHari').value;

            dbGuru.push({
                id: Date.now(),
                nama: nama,
                nip: nip,
                jk: jk,
                hari: hari
            });

            saveData();
            renderGuruList();
            renderPiketToday();
            populateGuruPiketSelect();
            this.reset();
            showToast("Guru " + nama + " berhasil ditambahkan.");
        };

        function renderGuruList() {
            const tbody = document.getElementById('tabelGuruAdmin');
            const noData = document.getElementById('noDataAdmin');
            
            if (dbGuru.length === 0) {
                tbody.innerHTML = "";
                noData.classList.remove('hidden');
                return;
            }
            
            noData.classList.add('hidden');
            tbody.innerHTML = dbGuru.map(g => `
                <tr class="border-b border-gray-50 hover:bg-gray-50 transition-all">
                    <td class="px-4 py-3">
                        <div class="font-bold text-indigo-900">${g.nama}</div>
                        <div class="text-[10px] text-gray-400 font-mono">NIP: ${g.nip} | ${g.hari}</div>
                    </td>
                    <td class="px-4 py-3 text-center">
                        <button onclick="hapusGuru(${g.id})" class="w-8 h-8 rounded-lg text-red-400 hover:bg-red-50 hover:text-red-600 transition flex items-center justify-center mx-auto">
                            <i class="fas fa-trash-alt"></i>
                        </button>
                    </td>
                </tr>
            `).join('');
        }

        function hapusGuru(id) {
            if(confirm("Hapus data guru?")) {
                dbGuru = dbGuru.filter(g => g.id !== id);
                saveData();
                renderGuruList();
                renderPiketToday();
                populateGuruPiketSelect();
                showToast("Data guru dihapus.");
            }
        }

        function renderPiketToday() {
            const days = ["Minggu", "Senin", "Selasa", "Rabu", "Kamis", "Jumat", "Sabtu"];
            const today = days[new Date().getDay()];
            const container = document.getElementById('listPiketToday');
            const pikel = dbGuru.filter(g => g.hari === today);
            
            container.innerHTML = pikel.length ? pikel.map(g => `
                <li class="flex items-center space-x-3 p-3 rounded-xl hover:bg-white hover:shadow-sm transition">
                    <div class="w-10 h-10 bg-indigo-100 rounded-full flex items-center justify-center text-indigo-700 font-bold border-2 border-white">
                        ${g.nama.split(' ').pop().charAt(0)}
                    </div>
                    <div>
                        <div class="font-bold text-xs text-indigo-900 leading-tight">${g.nama}</div>
                        <div class="text-[10px] text-gray-400">NIP: ${g.nip}</div>
                    </div>
                </li>
            `).join('') : `<li class="text-gray-400 text-[11px] italic text-center py-4">Tidak ada jadwal untuk hari ${today}</li>`;
        }

        function populateGuruPiketSelect() {
            const select = document.getElementById('izinGuruPiket');
            if (dbGuru.length === 0) {
                select.innerHTML = `<option disabled selected>Belum ada guru terdaftar</option>`;
                return;
            }
            select.innerHTML = dbGuru.map(g => `<option value="${g.id}">${g.nama} (NIP: ${g.nip})</option>`).join('');
        }

        function saveData() { localStorage.setItem('dbGuru', JSON.stringify(dbGuru)); }

        // --- LOGIKA SURAT IZIN ---
        function generateSurat() {
            const guruId = document.getElementById('izinGuruPiket').value;
            const namaSiswa = document.getElementById('izinNama').value;
            const kelas = document.getElementById('izinKelas').value;
            const tipe = document.getElementById('izinTipe').value;
            const alasan = document.getElementById('izinAlasan').value;
            const tglIzin = new Date(document.getElementById('izinTanggal').value);

            if(!namaSiswa || !kelas || !guruId) {
                showToast("Lengkapi Nama, Kelas, dan Guru Piket!");
                return;
            }

            const selectedGuru = dbGuru.find(g => g.id == guruId);
            const tglFormatted = tglIzin.toLocaleDateString('id-ID', {day:'numeric', month:'long', year:'numeric'});

            const contentHtml = `
                <div style="text-align: center; margin-bottom: 30px;">
                    <h3 style="text-decoration: underline; margin-bottom: 5px; font-weight: bold; font-size: 18px;">SURAT IZIN ${tipe.toUpperCase()}</h3>
                    <p style="font-size: 12px;">Nomor: ${Date.now()}/SMANCP-PIKET/${tglIzin.getFullYear()}</p>
                </div>
                <div style="margin-bottom: 20px;">
                    <p style="text-indent: 40px; margin-bottom: 15px;">Bersama dengan surat ini, Guru Piket ${schoolData.sekolah} memberikan izin kepada:</p>
                    <table style="width: 100%; margin-left: 20px; border-collapse: collapse;">
                        <tr><td style="width: 140px; padding: 4px 0;">Nama Siswa</td><td style="padding: 4px 0;">: <strong>${namaSiswa.toUpperCase()}</strong></td></tr>
                        <tr><td style="padding: 4px 0;">Kelas</td><td style="padding: 4px 0;">: ${kelas}</td></tr>
                        <tr><td style="padding: 4px 0;">Jenis Izin</td><td style="padding: 4px 0;">: <span style="border: 1px solid #000; padding: 1px 10px;">IZIN ${tipe.toUpperCase()}</span></td></tr>
                        <tr><td style="vertical-align: top; padding: 4px 0;">Keperluan / Alasan</td><td style="padding: 4px 0;">: ${alasan || 'Penting / Mendesak'}</td></tr>
                    </table>
                    <p style="margin-top: 20px; text-indent: 40px;">Demikian surat izin ini dibuat untuk dapat dipergunakan sebagaimana mestinya di lingkungan sekolah.</p>
                </div>
                <p style="text-align: right; font-size: 13px;">${schoolData.kota}, ${tglFormatted}</p>
            `;

            // Update PDF Kop & TTD Info
            document.getElementById('printKopInstansi1').innerText = schoolData.instansi1;
            document.getElementById('printKopInstansi2').innerText = schoolData.instansi2;
            document.getElementById('printKopSekolah').innerText = schoolData.sekolah;
            document.getElementById('printKopAlamat').innerText = schoolData.alamat;
            document.getElementById('printLabelTtdKota').innerText = `${schoolData.kota}, ${tglFormatted}`;

            // Update UI Previews
            document.getElementById('previewSurat').innerHTML = contentHtml;
            document.getElementById('printContent').innerHTML = contentHtml;
            document.getElementById('printNamaSiswa').innerText = `( ${namaSiswa.toUpperCase()} )`;
            document.getElementById('printTtdGuru').innerText = `( ${selectedGuru.nama} )`;
            document.getElementById('printNipGuru').innerText = `NIP. ${selectedGuru.nip}`;
            document.getElementById('printWaktu').innerText = new Date().toLocaleString('id-ID');

            // Toggle Tampilan
            document.getElementById('previewSurat').classList.remove('hidden');
            document.getElementById('previewPlaceholder').classList.add('hidden');
            document.getElementById('btnPrintSurat').classList.remove('hidden');
            showToast("Draft surat izin siap diunduh.");
        }

        // DOWNLOAD PDF
        function downloadPDF() {
            const element = document.getElementById('printArea');
            const namaSiswa = document.getElementById('izinNama').value || 'Siswa';
            const tipe = document.getElementById('izinTipe').value || 'Izin';
            
            const opt = {
                margin:       10,
                filename:     `Surat_${tipe}_${namaSiswa.replace(/\s+/g, '_')}.pdf`,
                image:        { type: 'jpeg', quality: 0.98 },
                html2canvas:  { scale: 2 },
                jsPDF:        { unit: 'mm', format: 'a4', orientation: 'portrait' }
            };

            element.classList.remove('hidden');
            showToast("Memproses file PDF...");

            html2pdf().set(opt).from(element).save().then(() => {
                element.classList.add('hidden');
                showToast("PDF Berhasil diunduh!");
            }).catch(err => {
                showToast("Gagal mengunduh PDF.");
            });
        }

        // --- AUTH & UTILS ---
        function showLogin() { document.getElementById('loginModal').classList.remove('hidden'); }
        function closeLogin() { document.getElementById('loginModal').classList.add('hidden'); }
        
        document.getElementById('loginForm').onsubmit = (e) => {
            e.preventDefault();
            const u = document.getElementById('username').value;
            const p = document.getElementById('password').value;
            
            if(u === 'admin' && p === 'admin') {
                isAdmin = true;
                localStorage.setItem('smartPiketAuth', 'true');
                showToast("Selamat Datang, Admin!");
                closeLogin();
                setTimeout(() => showTab('admin'), 500);
                document.getElementById('loginBtn').classList.add('hidden');
                document.getElementById('logoutBtn').classList.remove('hidden');
            } else {
                showToast("Username atau password salah!");
            }
        };

        function logout() {
            localStorage.removeItem('smartPiketAuth');
            isAdmin = false;
            showTab('dashboard');
            document.getElementById('loginBtn').classList.remove('hidden');
            document.getElementById('logoutBtn').classList.add('hidden');
            showToast("Sesi admin berakhir.");
        }

        function showToast(msg) {
            const t = document.getElementById('toast');
            document.getElementById('toastMsg').innerText = msg;
            t.classList.remove('opacity-0', 'translate-y-4', 'pointer-events-none');
            t.classList.add('opacity-100', 'translate-y-0');
            setTimeout(() => {
                t.classList.add('opacity-0', 'translate-y-4', 'pointer-events-none');
                t.classList.remove('opacity-100', 'translate-y-0');
            }, 3000);
        }

        function playBell(type) { showToast("Bel dibunyikan: " + type.toUpperCase()); }
        function cariInval() { document.getElementById('hasilInval').classList.remove('hidden'); }
    </script>
</body>
</html>
