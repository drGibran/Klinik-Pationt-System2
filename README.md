# Klinik-Pationt-System2
import React, { useState, useEffect } from 'react';
import { Save, Download, Plus, Trash2, FileText, Users, Calendar, DollarSign, Clock, Search, Filter } from 'lucide-react';

const PatientDataManager = () => {
  const [patients, setPatients] = useState([]);
  const [currentPatient, setCurrentPatient] = useState({
    nama: '',
    umur: '',
    telp: '',
    alamat: '',
    jenisLayanan: 'Konsultasi',
    biaya: '',
    metodePembayaran: 'Cash',
    keterangan: ''
  });
  
  const [selectedDate, setSelectedDate] = useState(new Date().toISOString().split('T')[0]);
  const [searchTerm, setSearchTerm] = useState('');
  const [filterService, setFilterService] = useState('All');
  const [patientCounter, setPatientCounter] = useState(1);

  // Load data from localStorage on component mount
  useEffect(() => {
    const savedPatients = localStorage.getItem('patientData');
    const savedCounter = localStorage.getItem('patientCounter');
    
    if (savedPatients) {
      setPatients(JSON.parse(savedPatients));
    }
    if (savedCounter) {
      setPatientCounter(parseInt(savedCounter));
    }
  }, []);

  // Save to localStorage whenever patients data changes
  useEffect(() => {
    localStorage.setItem('patientData', JSON.stringify(patients));
    localStorage.setItem('patientCounter', patientCounter.toString());
  }, [patients, patientCounter]);

  const jenisLayananOptions = [
    'Konsultasi',
    'Konsultasi + Obat',
    'Obat Saja',
    'Pemeriksaan Lab',
    'Tindakan Medis',
    'Kontrol Rutin'
  ];

  const metodePembayaranOptions = [
    'Cash',
    'Transfer Bank',
    'E-Wallet',
    'Debit Card'
  ];

  const addPatient = () => {
    if (!currentPatient.nama || !currentPatient.biaya) {
      alert('Mohon isi minimal nama pasien dan biaya!');
      return;
    }

    const newPatient = {
      id: Date.now(),
      kode: `#${String(patientCounter).padStart(3, '0')}`,
      tanggal: selectedDate,
      waktu: new Date().toLocaleTimeString('id-ID'),
      ...currentPatient,
      biaya: parseFloat(currentPatient.biaya) || 0
    };

    setPatients([...patients, newPatient]);
    setPatientCounter(patientCounter + 1);
    
    // Reset form
    setCurrentPatient({
      nama: '',
      umur: '',
      telp: '',
      alamat: '',
      jenisLayanan: 'Konsultasi',
      biaya: '',
      metodePembayaran: 'Cash',
      keterangan: ''
    });
  };

  const removePatient = (id) => {
    setPatients(patients.filter(p => p.id !== id));
  };

  const formatCurrency = (value) => {
    return new Intl.NumberFormat('id-ID', {
      style: 'currency',
      currency: 'IDR',
      minimumFractionDigits: 0,
    }).format(value);
  };

  const formatDate = (dateString) => {
    return new Date(dateString).toLocaleDateString('id-ID', {
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    });
  };

  const todayPatients = patients.filter(p => p.tanggal === selectedDate);
  
  const filteredPatients = todayPatients.filter(p => {
    const matchesSearch = p.nama.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         p.kode.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesFilter = filterService === 'All' || p.jenisLayanan === filterService;
    return matchesSearch && matchesFilter;
  });

  const todayStats = {
    totalPatients: todayPatients.length,
    totalRevenue: todayPatients.reduce((sum, p) => sum + p.biaya, 0),
    avgRevenue: todayPatients.length > 0 ? todayPatients.reduce((sum, p) => sum + p.biaya, 0) / todayPatients.length : 0,
    serviceBreakdown: jenisLayananOptions.reduce((acc, service) => {
      acc[service] = todayPatients.filter(p => p.jenisLayanan === service).length;
      return acc;
    }, {})
  };

  const exportToPDF = () => {
    const printContent = `
      <html>
        <head>
          <title>Laporan Harian Klinik - ${formatDate(selectedDate)}</title>
          <style>
            body { 
              font-family: Arial, sans-serif; 
              margin: 20px; 
              font-size: 12px;
            }
            .header { 
              text-align: center; 
              margin-bottom: 30px; 
              border-bottom: 2px solid #333;
              padding-bottom: 20px;
            }
            .stats { 
              display: flex; 
              justify-content: space-around; 
              margin-bottom: 20px;
              background-color: #f5f5f5;
              padding: 15px;
              border-radius: 5px;
            }
            .stats-item {
              text-align: center;
            }
            .stats-title {
              font-weight: bold;
              color: #666;
              font-size: 10px;
            }
            .stats-value {
              font-size: 16px;
              font-weight: bold;
              color: #333;
            }
            .patient-table { 
              width: 100%; 
              border-collapse: collapse; 
              margin-bottom: 20px;
            }
            .patient-table th, .patient-table td { 
              border: 1px solid #ddd; 
              padding: 8px; 
              text-align: left; 
              font-size: 11px;
            }
            .patient-table th { 
              background-color: #4F46E5; 
              color: white;
              font-weight: bold; 
            }
            .patient-table tr:nth-child(even) {
              background-color: #f9f9f9;
            }
            .amount { 
              text-align: right; 
              font-weight: bold;
            }
            .total-row { 
              font-weight: bold; 
              background-color: #e8f4f8 !important;
            }
            .footer {
              margin-top: 30px;
              text-align: center;
              font-size: 10px;
              color: #666;
              border-top: 1px solid #ddd;
              padding-top: 15px;
            }
            .summary-box {
              background-color: #f0f9ff;
              border: 1px solid #3b82f6;
              padding: 15px;
              margin: 20px 0;
              border-radius: 5px;
            }
            .kode {
              background-color: #e0e7ff;
              padding: 2px 6px;
              border-radius: 3px;
              font-family: monospace;
              font-weight: bold;
            }
          </style>
        </head>
        <body>
          <div class="header">
            <h2>LAPORAN HARIAN PASIEN</h2>
            <h3>KLINIK SERUNI HUSADA</h3>
            <p>Tanggal: ${formatDate(selectedDate)}</p>
            <p>Dicetak: ${new Date().toLocaleDateString('id-ID')} ${new Date().toLocaleTimeString('id-ID')}</p>
          </div>
          
          <div class="stats">
            <div class="stats-item">
              <div class="stats-title">TOTAL PASIEN</div>
              <div class="stats-value">${todayStats.totalPatients}</div>
            </div>
            <div class="stats-item">
              <div class="stats-title">TOTAL PENDAPATAN</div>
              <div class="stats-value">${formatCurrency(todayStats.totalRevenue)}</div>
            </div>
            <div class="stats-item">
              <div class="stats-title">RATA-RATA/PASIEN</div>
              <div class="stats-value">${formatCurrency(todayStats.avgRevenue)}</div>
            </div>
          </div>

          <div class="summary-box">
            <h4>BREAKDOWN LAYANAN:</h4>
            ${Object.entries(todayStats.serviceBreakdown)
              .filter(([service, count]) => count > 0)
              .map(([service, count]) => `<p>• ${service}: ${count} pasien</p>`)
              .join('')}
          </div>
          
          <table class="patient-table">
            <thead>
              <tr>
                <th>No</th>
                <th>Kode</th>
                <th>Waktu</th>
                <th>Nama Pasien</th>
                <th>Umur</th>
                <th>Jenis Layanan</th>
                <th>Biaya</th>
                <th>Pembayaran</th>
                <th>Keterangan</th>
              </tr>
            </thead>
            <tbody>
              ${todayPatients.map((patient, index) => `
                <tr>
                  <td>${index + 1}</td>
                  <td><span class="kode">${patient.kode}</span></td>
                  <td>${patient.waktu}</td>
                  <td><strong>${patient.nama}</strong></td>
                  <td>${patient.umur}</td>
                  <td>${patient.jenisLayanan}</td>
                  <td class="amount">${formatCurrency(patient.biaya)}</td>
                  <td>${patient.metodePembayaran}</td>
                  <td>${patient.keterangan}</td>
                </tr>
              `).join('')}
              <tr class="total-row">
                <td colspan="6"><strong>TOTAL HARI INI</strong></td>
                <td class="amount"><strong>${formatCurrency(todayStats.totalRevenue)}</strong></td>
                <td colspan="2"><strong>${todayStats.totalPatients} PASIEN</strong></td>
              </tr>
            </tbody>
          </table>

          <div class="summary-box">
            <h4>TEMPLATE JURNAL UMUM EXCEL:</h4>
            ${todayPatients.map((patient, index) => `
              <p><strong>Row ${index + 1}:</strong> Tanggal: ${patient.tanggal} | Kategori: Pendapatan | 
              Akun: Pendapatan Konsultasi | Bank: Kas | 
              Deskripsi: ${patient.jenisLayanan} - ${patient.nama} ${patient.kode} | 
              Debet: ${patient.biaya}</p>
            `).join('')}
          </div>
          
          <div class="footer">
            <p><strong>Sistem Manajemen Pasien Klinik</strong></p>
            <p>Generated automatically • Keep this document for daily records</p>
          </div>
        </body>
      </html>
    `;

    const printWindow = window.open('', '_blank');
    printWindow.document.write(printContent);
    printWindow.document.close();
    printWindow.focus();
    printWindow.print();
  };

  const exportToExcelFormat = () => {
    const excelData = todayPatients.map((patient, index) => ({
      'No': index + 1,
      'Tanggal': patient.tanggal,
      'Bulan': new Date(patient.tanggal).toLocaleDateString('en-US', { month: 'long' }),
      'Tahun': new Date(patient.tanggal).getFullYear(),
      'Kategori': 'Pendapatan',
      'Nama Akun': 'Pendapatan Konsultasi',
      'Bank': 'Kas',
      'Deskripsi': `${patient.jenisLayanan} - ${patient.nama} ${patient.kode}`,
      'Debet': patient.biaya,
      'Kredit': '',
      'Kode Pasien': patient.kode,
      'Nama Pasien': patient.nama,
      'Jenis Layanan': patient.jenisLayanan
    }));

    const csvContent = [
      Object.keys(excelData[0]).join(','),
      ...excelData.map(row => Object.values(row).join(','))
    ].join('\n');

    const blob = new Blob([csvContent], { type: 'text/csv' });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `klinik-data-${selectedDate}.csv`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    window.URL.revokeObjectURL(url);
  };

  return (
    <div className="max-w-7xl mx-auto p-6 bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen">
      {/* Header */}
      <div className="text-center mb-8">
        <div className="flex items-center justify-center gap-3 mb-4">
          <Users className="w-10 h-10 text-indigo-600" />
          <h1 className="text-3xl font-bold text-gray-800">Sistem Manajemen Pasien Klinik</h1>
        </div>
        <p className="text-gray-600 text-lg">Daily Patient Recording & Analytics Dashboard</p>
      </div>

      {/* Date Selection & Stats */}
      <div className="bg-white rounded-xl shadow-lg p-6 mb-6">
        <div className="flex items-center justify-between mb-6">
          <div className="flex items-center gap-4">
            <Calendar className="w-6 h-6 text-indigo-600" />
            <label className="block text-sm font-medium text-gray-700">Tanggal Pelayanan:</label>
            <input
              type="date"
              value={selectedDate}
              onChange={(e) => setSelectedDate(e.target.value)}
              className="p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
            />
          </div>
          <div className="flex gap-2">
            <button
              onClick={exportToPDF}
              className="bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700 transition-colors font-medium flex items-center gap-2"
            >
              <Download className="w-4 h-4" />
              Export PDF
            </button>
            <button
              onClick={exportToExcelFormat}
              className="bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition-colors font-medium flex items-center gap-2"
            >
              <FileText className="w-4 h-4" />
              Export Excel
            </button>
          </div>
        </div>

        {/* Daily Stats */}
        <div className="grid md:grid-cols-4 gap-4 mb-6">
          <div className="bg-blue-50 p-4 rounded-lg text-center">
            <Users className="w-8 h-8 text-blue-600 mx-auto mb-2" />
            <div className="text-2xl font-bold text-blue-800">{todayStats.totalPatients}</div>
            <div className="text-sm text-blue-600">Total Pasien</div>
          </div>
          <div className="bg-green-50 p-4 rounded-lg text-center">
            <DollarSign className="w-8 h-8 text-green-600 mx-auto mb-2" />
            <div className="text-2xl font-bold text-green-800">{formatCurrency(todayStats.totalRevenue)}</div>
            <div className="text-sm text-green-600">Total Pendapatan</div>
          </div>
          <div className="bg-purple-50 p-4 rounded-lg text-center">
            <Clock className="w-8 h-8 text-purple-600 mx-auto mb-2" />
            <div className="text-2xl font-bold text-purple-800">{formatCurrency(todayStats.avgRevenue)}</div>
            <div className="text-sm text-purple-600">Rata-rata/Pasien</div>
          </div>
          <div className="bg-orange-50 p-4 rounded-lg text-center">
            <FileText className="w-8 h-8 text-orange-600 mx-auto mb-2" />
            <div className="text-2xl font-bold text-orange-800">#{String(patientCounter).padStart(3, '0')}</div>
            <div className="text-sm text-orange-600">Kode Berikutnya</div>
          </div>
        </div>
      </div>

      <div className="grid lg:grid-cols-3 gap-6">
        {/* Input Form */}
        <div className="lg:col-span-1">
          <div className="bg-white rounded-xl shadow-lg p-6">
            <h2 className="text-xl font-bold text-gray-800 mb-4 flex items-center gap-2">
              <Plus className="w-5 h-5" />
              Input Pasien Baru
            </h2>
            
            <div className="space-y-4">
              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">Nama Pasien *</label>
                <input
                  type="text"
                  value={currentPatient.nama}
                  onChange={(e) => setCurrentPatient({...currentPatient, nama: e.target.value})}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                  placeholder="Contoh: Tn. Ahmad"
                />
              </div>

              <div className="grid grid-cols-2 gap-3">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Umur</label>
                  <input
                    type="text"
                    value={currentPatient.umur}
                    onChange={(e) => setCurrentPatient({...currentPatient, umur: e.target.value})}
                    className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                    placeholder="25"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Biaya *</label>
                  <input
                    type="number"
                    value={currentPatient.biaya}
                    onChange={(e) => setCurrentPatient({...currentPatient, biaya: e.target.value})}
                    className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                    placeholder="50000"
                  />
                </div>
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">No. Telepon</label>
                <input
                  type="text"
                  value={currentPatient.telp}
                  onChange={(e) => setCurrentPatient({...currentPatient, telp: e.target.value})}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                  placeholder="08123456789"
                />
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">Alamat</label>
                <input
                  type="text"
                  value={currentPatient.alamat}
                  onChange={(e) => setCurrentPatient({...currentPatient, alamat: e.target.value})}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                  placeholder="Jl. Merdeka No.5"
                />
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">Jenis Layanan</label>
                <select
                  value={currentPatient.jenisLayanan}
                  onChange={(e) => setCurrentPatient({...currentPatient, jenisLayanan: e.target.value})}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                >
                  {jenisLayananOptions.map(option => (
                    <option key={option} value={option}>{option}</option>
                  ))}
                </select>
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">Metode Pembayaran</label>
                <select
                  value={currentPatient.metodePembayaran}
                  onChange={(e) => setCurrentPatient({...currentPatient, metodePembayaran: e.target.value})}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                >
                  {metodePembayaranOptions.map(option => (
                    <option key={option} value={option}>{option}</option>
                  ))}
                </select>
              </div>

              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">Keterangan</label>
                <textarea
                  value={currentPatient.keterangan}
                  onChange={(e) => setCurrentPatient({...currentPatient, keterangan: e.target.value})}
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                  placeholder="Catatan tambahan..."
                  rows="2"
                />
              </div>

              <button
                onClick={addPatient}
                className="w-full bg-indigo-600 text-white px-6 py-3 rounded-lg hover:bg-indigo-700 transition-colors font-medium flex items-center justify-center gap-2"
              >
                <Plus className="w-5 h-5" />
                Tambah Pasien
              </button>
            </div>
          </div>
        </div>

        {/* Patient List */}
        <div className="lg:col-span-2">
          <div className="bg-white rounded-xl shadow-lg p-6">
            <div className="flex items-center justify-between mb-4">
              <h2 className="text-xl font-bold text-gray-800">
                Daftar Pasien - {formatDate(selectedDate)}
              </h2>
              <div className="flex gap-3">
                <div className="relative">
                  <Search className="w-5 h-5 absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400" />
                  <input
                    type="text"
                    value={searchTerm}
                    onChange={(e) => setSearchTerm(e.target.value)}
                    className="pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                    placeholder="Cari nama/kode..."
                  />
                </div>
                <select
                  value={filterService}
                  onChange={(e) => setFilterService(e.target.value)}
                  className="px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                >
                  <option value="All">Semua Layanan</option>
                  {jenisLayananOptions.map(option => (
                    <option key={option} value={option}>{option}</option>
                  ))}
                </select>
              </div>
            </div>

            {filteredPatients.length === 0 ? (
              <div className="text-center py-8 text-gray-500">
                <Users className="w-16 h-16 mx-auto mb-4 text-gray-300" />
                <p>Belum ada data pasien untuk tanggal ini</p>
              </div>
            ) : (
              <div className="overflow-x-auto">
                <table className="w-full border-collapse">
                  <thead>
                    <tr className="border-b-2 border-gray-200">
                      <th className="text-left p-3 font-semibold text-gray-700">Kode</th>
                      <th className="text-left p-3 font-semibold text-gray-700">Waktu</th>
                      <th className="text-left p-3 font-semibold text-gray-700">Nama</th>
                      <th className="text-left p-3 font-semibold text-gray-700">Layanan</th>
                      <th className="text-right p-3 font-semibold text-gray-700">Biaya</th>
                      <th className="text-center p-3 font-semibold text-gray-700">Aksi</th>
                    </tr>
                  </thead>
                  <tbody>
                    {filteredPatients.map((patient, index) => (
                      <tr key={patient.id} className="border-b border-gray-100 hover:bg-gray-50">
                        <td className="p-3">
                          <span className="bg-indigo-100 text-indigo-800 px-2 py-1 rounded text-sm font-mono">
                            {patient.kode}
                          </span>
                        </td>
                        <td className="p-3 text-sm text-gray-600">{patient.waktu}</td>
                        <td className="p-3">
                          <div>
                            <div className="font-medium">{patient.nama}</div>
                            {patient.umur && <div className="text-sm text-gray-500">{patient.umur} tahun</div>}
                          </div>
                        </td>
                        <td className="p-3">
                          <span className="bg-green-100 text-green-800 px-2 py-1 rounded text-sm">
                            {patient.jenisLayanan}
                          </span>
                        </td>
                        <td className="p-3 text-right font-medium">{formatCurrency(patient.biaya)}</td>
                        <td className="p-3 text-center">
                          <button
                            onClick={() => removePatient(patient.id)}
                            className="text-red-600 hover:text-red-800 p-1"
                          >
                            <Trash2 className="w-4 h-4" />
                          </button>
                        </td>
                      </tr>
                    ))}
                  </tbody>
                  <tfoot>
                    <tr className="border-t-2 border-gray-200 bg-gray-50">
                      <td colSpan="4" className="p-3 font-bold">TOTAL HARI INI</td>
                      <td className="p-3 text-right font-bold">{formatCurrency(todayStats.totalRevenue)}</td>
                      <td className="p-3 text-center font-bold">{todayStats.totalPatients} pasien</td>
                    </tr>
                  </tfoot>
                </table>
              </div>
            )}
          </div>
        </div>
      </div>

      {/* Footer */}
      <div className="mt-8 text-center text-gray-500 text-sm">
        <p>💡 <strong>Data otomatis tersimpan di browser</strong> - Export PDF/Excel untuk backup</p>
        <p className="mt-1">📊 Copy data Excel format untuk input ke Jurnal Umum dengan mudah</p>
      </div>
    </div>
  );
};

export default PatientDataManager;
