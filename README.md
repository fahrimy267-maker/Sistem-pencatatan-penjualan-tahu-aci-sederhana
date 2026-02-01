#Sistem-pencatatan-penjualan-tahu-aci-sederhana
#Program ini adalah sebuah sistem pencatatan kasir sederhana yang bisa menghitung penjualan, keuntungan, dan merekap data penjualan. berbasis terminal 
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime
conn = sqlite3.connect('tahu_aci.db')
cursor = conn.cursor()
cursor.execute('''
CREATE TABLE IF NOT EXISTS transaksi (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    tanggal TEXT,
    menu TEXT,
    pcs INTEGER,
    harga INTEGER,
    total INTEGER
)
''')
cursor.execute('''
CREATE TABLE IF NOT EXISTS laporan (
    tanggal TEXT PRIMARY KEY,
    stok_awal INTEGER,
    stok_sisa INTEGER,
    modal INTEGER,
    pendapatan INTEGER,
    keuntungan INTEGER
)
''')
conn.commit()
def kasir():
    cursor.execute('SELECT stok_sisa FROM laporan WHERE tanggal=?', (hari,))
    stok = cursor.fetchone()[0]
    total_bayar = 0
    pesanan = []

    while True:
        print('=== MENU TAHU ACI ===')
        print('1. 6 pcs   | Rp 10.000')
        print('2. 10 pcs  | Rp 15.000')
        print('3. 14 pcs  | Rp 20.000')

        pilih = input('Masukan Menu [1-3] : ')
        if pilih not in menu:
            print('Menu tidak valid')
            continue
        try :
            jumlah = int(input('Masukan Jumlah : '))
            if jumlah <= 0 :
                print('Jumlah harus lebih dari 0 ')
                continue
        except ValueError :
            print('jumlah harus berupa angka ')
            continue
        nama, pcs, harga = menu[pilih]
        total_pcs = pcs * jumlah
        if total_pcs > stok:
            print(f'Stok tidak cukup. Sisa stok : {stok}')
            continue

        total = harga * jumlah
        stok -= total_pcs
        total_bayar += total
        pesanan.append((nama, jumlah, total))
        cursor.execute('''
        INSERT INTO transaksi
        (tanggal, menu, pcs, harga, total)
        VALUES (?, ?, ?, ?, ?)
        ''', (hari, nama, total_pcs, harga, total))
        cursor.execute(
        'UPDATE laporan SET stok_sisa=? WHERE tanggal=?',
        (stok, hari)
        )
        cursor.execute('UPDATE laporan SET stok_sisa=? WHERE tanggal=?', (stok, hari))
        conn.commit()
        print('--- Detail Pesanan ---')
        print(f'Pesanan  : {nama}')
        print(f'Jumlah   : {jumlah} x')
        print(f'Subtotal : Rp. {total:,}')
        print(f'Total    : Rp. {total_bayar:,}')
        print(f'Stok sisa: {stok}')
        tambah = input('Tambah pesanan? [y/t] : ').lower()
        if tambah == 't':
            break

    while True :
      try :
        bayar = int(input('Masukan Pembayaran : '))
        if bayar < total_bayar :
          print('uang tidak cukup')
          continue
        break
      except ValueError :
          print('pembayaran harus berupa angka ')
    kembali = bayar - total_bayar
    print('=== STRUK PEMBAYARAN ===')
    for i in pesanan:
        print(f'{i[0]} x {i[1]} = Rp {i[2]:,}')
    print(f'TOTAL BAYAR : Rp {total_bayar:,}')
    print(f'PEMBAYARAN  : Rp {bayar:,}')
    print(f'KEMBALIAN   : Rp {kembali:,}')
    print('*** TERIMA KASIH ***')
    print('======================')
    print(f'SISA STOK   : {stok}')
def hitung_modal():
    df_pendapatan = pd.read_sql('SELECT SUM(total) AS total FROM transaksi WHERE tanggal=?', conn, params=(hari,))
    pendapatan = df_pendapatan['total'][0]
    pendapatan = int(pendapatan) if pendapatan is not None else 0
    cursor.execute('SELECT modal FROM laporan WHERE tanggal=?', (hari,))
    modal = cursor.fetchone()[0]
    untung = pendapatan - modal
    cursor.execute('''
        UPDATE laporan
        SET pendapatan = ?, keuntungan = ?
        WHERE tanggal = ?
        ''', (pendapatan, untung, hari))
    conn.commit()


    print(f'Pendapatan Hari Ini : Rp {pendapatan:,}')
    print(f'Keuntungan Hari Ini : Rp {untung:,}')
def tampilkan_transaksi():
    print('*** DATA TRANSAKSI ***')
    df1 = pd.read_sql('SELECT * FROM transaksi', conn)
    print(df1)
def tampilkan_laporan():
    print('*** DATA LAPORAN HARIAN ***')
    df2 = pd.read_sql("SELECT * FROM laporan", conn)
    print(df2)
def grafik_keuntungan():
    df = pd.read_sql('SELECT tanggal, keuntungan FROM laporan', conn)
    if df.empty:
        print('Belum ada data grafik.')
    else:
        plt.figure(figsize=(8, 5))
        plt.bar(df['tanggal'], df['keuntungan'])
        plt.title('Grafik Keuntungan Harian')
        plt.xlabel('Tanggal')
        plt.ylabel('Keuntungan (Rp)')
        plt.show()

menu = {
    '1': ('6 pcs', 6, 10000),
    '2': ('10 pcs', 10, 15000),
    '3': ('14 pcs', 14, 20000)
}

hari = datetime.now().strftime('%Y-%m-%d')
cursor.execute('SELECT stok_sisa FROM laporan WHERE tanggal=?', (hari,))
cek = cursor.fetchone()

if not cek:
    print('=== INPUT DATA HARIAN ===')
    stok = int(input('Masukan Stok Hari Ini  : '))
    modal = int(input('Masukan Modal Hari Ini : '))
    cursor.execute('''
    INSERT INTO laporan
    (tanggal, stok_awal, stok_sisa, modal, pendapatan, keuntungan)
    VALUES (?, ?, ?, ?, 0, 0)
    ''', (hari, stok, stok, modal))
    conn.commit()
else:
    stok = cek[0]
while True:
    print('=== TAHU ACI TEGAL MAS AJI ===')
    print('+++ Menu Utama +++')
    print('1. Kasir')
    print('2. Perhitungan Modal')
    print('3. Tampilkan Laporan Transaksi')
    print('4. Tampilkan Laporan Keuangan')
    print('5. Tampilkan Grafik Keuntungan')
    print('0. Keluar')
    pilih_menu = input('Pilih Menu : ')
    if pilih_menu == '1':
       kasir()
    elif pilih_menu == '2':
        hitung_modal()
    elif pilih_menu == '3':
        tampilkan_transaksi()
    elif pilih_menu == '4':
        tampilkan_laporan()
    elif pilih_menu == '5':
        grafik_keuntungan()
    elif pilih_menu == '0':
        conn.close()
        print('Program OFF')
        break
    else:
        print('Menu tidak valid')
