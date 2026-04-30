# Tugas-1-
Kecerdasan Komputasional
import random
import matplotlib.pyplot as plt

    #= DATA PENGELUARAN (Dataset) =#
# Daftar barang: (Nama, Harga, Nilai Kepuasan)
    items = [
    ("Makan", 20000, 80),
    ("Kopi", 15000, 60),
    ("Jajan", 10000, 50),
    ("Transport", 10000, 70),
    ("Internet", 5000, 40),
]

    BUDGET = 50000  # Batas maksimal uang yang boleh dikeluarkan

# Parameter Algoritma Genetika

    POP_SIZE = 30       # Jumlah kandidat solusi dalam satu populasi
    GENERATIONS = 100   # Berapa kali proses evolusi diulang
    MUTATION_RATE = 0.2 # Peluang mutasi gen (20%)

    #= INIT (Inisialisasi)=#
# Membuat satu individu (solusi) secara acak dalam bentuk biner [0, 1, ...]
    def create_individual():
    return [random.randint(0,1) for _ in items]
#
Pada tahap ini, program membuat "populasi awal" yang berisi sekumpulan individu. Setiap individu direpresentasikan dalam bentuk kode biner (angka 0 dan 1). Angka 1 berarti barang tersebut dibeli, dan 0 berarti tidak. Karena ini adalah langkah awal, semua kombinasi dibuat secara acak. Tujuannya adalah untuk memiliki titik awal yang beragam, sehingga algoritma memiliki banyak variasi solusi untuk dicoba dan dikembangkan.


# Membuat kumpulan individu (populasi) sebanyak POP_SIZE
    def init_population():
    return [create_individual() for _ in range(POP_SIZE)]

    #= FITNESS (Fungsi Penilaian)=#
# Menghitung seberapa baik sebuah solusi
    def fitness(ind):
    total_cost = 0
    total_value = 0

    for gene, (name, cost, value) in zip(ind, items):
        if gene == 1: # Jika barang dipilih
            total_cost += cost
            total_value += value

    # Jika melebihi budget, nilai fitness nol (solusi buruk/tidak valid)
    if total_cost > BUDGET:
        return 0

    return total_value # Fitness adalah total kepuasan
#
Fungsi fitness adalah jantung dari algoritma ini karena berfungsi sebagai "juri" yang menilai kualitas setiap solusi. Program akan menjumlahkan total harga dan total kepuasan dari barang-barang yang dipilih oleh satu individu. Jika total harganya melebihi budget (Rp50.000), maka nilai fitness-nya langsung dianggap nol atau sangat buruk. Sebaliknya, jika masuk dalam budget, nilai fitness-nya adalah total kepuasannya. Semakin tinggi kepuasannya, semakin dianggap "layak" solusi tersebut untuk dipertahankan.
    #= SELECTION (Seleksi)=#
# Memilih induk terbaik menggunakan metode Tournament (ambil 3 acak, pilih 1 terbaik)
    def selection(pop):
    return max(random.sample(pop, 3), key=fitness)
#
Seleksi meniru hukum alam di mana individu yang kuat lebih berpeluang bertahan hidup. Di sini digunakan metode Tournament Selection, yaitu dengan mengambil tiga individu secara acak lalu memilih yang terbaik di antara mereka untuk dijadikan orang tua. Proses ini memastikan bahwa solusi-solusi yang memiliki nilai kepuasan tinggi memiliki kesempatan lebih besar untuk menurunkan "gen" atau karakteristik mereka ke generasi berikutnya.
    #= CROSSOVER (Perkawinan Silang)=#
# Menggabungkan gen dari dua induk untuk membuat anak (solusi baru)
    def crossover(p1, p2):
    point = random.randint(1, len(p1)-1) # Titik potong acak
    return p1[:point] + p2[point:]
#
Setelah orang tua terpilih, dilakukan proses crossover atau perkawinan silang untuk menghasilkan keturunan (solusi baru). Cara kerjanya adalah dengan memotong bagian gen dari ayah dan menggabungkannya dengan gen dari ibu pada titik tertentu. Dengan cara ini, anak yang dihasilkan mewarisi sebagian sifat baik dari kedua induknya. Ini adalah langkah utama untuk mengeksplorasi kombinasi solusi baru yang mungkin lebih optimal dari sebelumnya.
    #= MUTATION (Mutasi)=#
# Memberikan perubahan acak pada gen agar variasi solusi tetap terjaga
    def mutate(ind):
    for i in range(len(ind)):
        if random.random() < MUTATION_RATE:
            ind[i] = 1 - ind[i] # Tukar biner (0 jadi 1 atau sebaliknya)
    return ind
#
Mutasi adalah perubahan kecil yang terjadi secara acak pada gen anak, seperti mengubah angka 0 menjadi 1 atau sebaliknya. Meskipun terlihat merusak, mutasi sangat penting untuk menjaga variasi dalam populasi agar algoritma tidak terjebak pada solusi yang itu-itu saja (stagnan). Mutasi memungkinkan algoritma untuk "mencoba hal baru" yang mungkin tidak ada pada gen orang tuanya, sehingga mencegah hasil akhir yang membosankan atau kurang optimal.
    #= DECODE (Penerjemah Solusi)=#
# Mengubah kode biner menjadi informasi yang bisa dibaca manusia
    def decode(ind):
    chosen = []
    total_cost = 0
    total_value = 0

    for gene, (name, cost, value) in zip(ind, items):
        if gene == 1:
            chosen.append(name)
            total_cost += cost
            total_value += value

    return chosen, total_cost, total_value

    #= GA (Genetic Algorithm Main Loop)=#
# Fungsi utama untuk menjalankan simulasi evolusi
    def GA():
    pop = init_population() # Inisialisasi populasi awal
    best_hist = []          # Untuk menyimpan riwayat perkembangan kepuasan
#
Program menjalankan proses seleksi, perkawinan, dan mutasi secara berulang-ulang hingga 100 generasi. Di setiap generasinya, terdapat konsep Elitisme, di mana dua individu terbaik selalu dipertahankan tanpa diubah sedikit pun agar solusi terbaik yang sudah ditemukan tidak hilang secara tidak sengaja. Seiring berjalannya generasi, rata-rata kualitas solusi dalam populasi akan terus meningkat hingga mencapai titik yang dianggap paling optimal.
    print("\n===== OPTIMASI UANG JAJAN =====\n")

    for gen in range(GENERATIONS):
        # Urutkan populasi dari yang terbaik berdasarkan fitness
        pop = sorted(pop, key=fitness, reverse=True)

        best = pop[0]
        best_fit = fitness(best)
        best_hist.append(best_fit)

        if gen % 5 == 0:
            print(f"Gen {gen:3d} | Nilai kepuasan: {best_fit}")

        # Elitisme: Pertahankan 2 solusi terbaik agar tidak hilang
        new_pop = pop[:2]

        # Isi sisa populasi baru dengan keturunan baru
        while len(new_pop) < POP_SIZE:
            p1 = selection(pop) # Pilih Ayah
            p2 = selection(pop) # Pilih Ibu

            child = crossover(p1, p2) # Kawinkan
            child = mutate(child)     # Mutasi acak

            new_pop.append(child)

        pop = new_pop # Perbarui populasi ke generasi berikutnya

    # --- HASIL AKHIR ---
    best = sorted(pop, key=fitness, reverse=True)[0]
    chosen, cost, value = decode(best)

    print("\n===== HASIL AKHIR =====")
    print("Pilihan:", chosen)
    print("Total biaya:", cost)
    print("Total kepuasan:", value)

    if cost <= BUDGET:
        print("Penjelasan: Pengeluaran optimal sesuai budget.")
    else:
        print("Penjelasan: Melebihi budget (tidak valid).")
#
Setelah semua  selesai, program akan mengambil individu terbaik yang pernah ditemukan sepanjang proses evolusi. Kode kemudian menerjemahkan kembali angka biner menjadi nama barang yang nyata, menghitung total biaya, dan menampilkan total kepuasannya. Terakhir, grafik dari Matplotlib akan menunjukkan grafik perkembangan fitness, di mana kita biasanya akan melihat garis yang naik secara bertahap, membuktikan bahwa algoritma berhasil belajar mencari solusi yang lebih baik dari waktu ke waktu.
    # --- VISUALISASI ---
    plt.figure()
    plt.plot(best_hist)
    plt.title("Perkembangan Kepuasan")
    plt.xlabel("Generasi")
    plt.ylabel("Fitness")
    plt.grid()
    plt.show()

# Jalankan Program
    GA()
#
Fungsi GA() adalah mesin penggerak utama yang mensimulasikan proses evolusi untuk menemukan daftar belanja terbaik. Di dalamnya, program secara berulang menilai setiap pilihan barang, mengamankan kombinasi yang paling memuaskan (elitisme), dan mengawin-silangkan ide-ide bagus untuk menciptakan kombinasi baru yang lebih efektif. Melalui proses seleksi dan mutasi yang dilakukan berkali-kali, fungsi ini secara otomatis menyaring ribuan kemungkinan hingga menyisakan satu rekomendasi final yang paling memberikan kepuasan maksimal tanpa melampaui batas budget Rp50.000.
