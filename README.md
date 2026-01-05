# FPGA-Based FIR Filter Design

## Laporan Tugas Besar  
### Desain FPGA dan SoC 2025

**Kelompok** : 5  

**Nama–NIM Anggota 1** : MUHAMMAD IRFAN ZIDNY / 1102223172  
**Nama–NIM Anggota 2** : FARHAN RUSYDAN ARIEF / 1102223164  
**Nama–NIM Anggota 3** : MOH ELDIVO ALSYAWAL OTOLUWA / 1102223205  

---

## Judul  

**FPGA-BASED FIR FILTER DESIGN**

---

## Deskripsi  

Tugas besar ini merancang sistem pemrosesan sinyal digital berupa Filter FIR (Finite Impulse Response) 4-Tap pada FPGA. Sistem ini mengimplementasikan arsitektur hardware yang dikontrol oleh Finite State Machine (FSM) untuk mengatur aliran data dan operasi aritmatika. FSM bertugas memastikan proses pengambilan sampel data (sampling) dan pergeseran register (shifting) berjalan sinkron dan stabil setiap kali pengguna memberikan perintah step.

---

## Fungsi  

1. **Penyaringan Sinyal**: Meredam perubahan data drastis (noise) menggunakan algoritma Moving Average.  
2. **Kontrol Sekuensial**: Mendemonstrasikan penggunaan State Machine untuk mengatur timing proses digital.  
3. **Visualisasi Data**: Menampilkan hasil filter secara real-time ke 7-Segment Display dalam format desimal.  

---

## Fitur dan Spesifikasi  

### Fitur  

| Fitur | Deskripsi |
|-----|----------|
| 4-Tap Moving Average Algorithm | Mampu meratakan fluktuasi sinyal dengan menghitung rata-rata dari 4 sampel data terakhir |
| Eksekusi sekuensial per step | Mode operasi manual yang memungkinkan pengguna mengamati perubahan data per satu siklus clock |
| Visualisasi 7-Segment | Hasil filter dapat diamati melalui 7 segment |
| Edge detection | Mampu mendeteksi input tombol sebagai trigger input dan pemrosesan data |

---

### Spesifikasi  

1. Platform Hardware: FPGA Altera Cyclone V (DE1-SoC Board).  
2. Clock Frequency: 50 MHz (System Clock).  
3. Controller: Finite State Machine (2-State: IDLE, SHIFT).  
4. Arsitektur Data:  
   - Input 8-bit  
   - Akumulator 10-bit  
   - Output 8-bit  
5. Metode Filter: 4-Tap Moving Average  
   y[n] = (x1+x2+...+x(n-1)+xn)/n
6. Input:  
   - KEY[0]: Reset Asinkron (Active Low).  
   - KEY[1]: Step Trigger (Active Low) dengan Edge Detection.  
   - SW[0]: Selector Input Data (10 atau 90).  

7. Output:  
   - 7-Segment (Nilai Filter)  
   - LEDR[7:0]: Monitor Data Input  
   - LEDR[9]: Button Press Indicator  

---

## Cara Penggunaan  

![Flowchart Pengguna](images/flowchart.png)

1. Sistem diaktifkan dengan memberikan catu daya pada board FPGA dan mengunggah program (bitstream).  
2. Sistem melakukan inisialisasi awal saat tombol Reset (KEY[0]) ditekan, mengembalikan seluruh data register dan tampilan ke nilai default (10).  
3. Pengguna menentukan target input dengan menggeser Switch (SW[0]) ke posisi bawah (nilai 10) atau atas (nilai 90).  
4. Lampu indikator LEDR[0–7] menyala seketika untuk memvisualisasikan data biner dari input yang dipilih.  
5. Tombol Step (KEY[1]) ditekan untuk memicu FSM mengambil satu sampel data baru.  
6. Lampu status LEDR[9] menyala sesaat selama tombol Step ditekan.  
7. Sistem memproses data secara internal dan menampilkan hasil rata-rata pada 7-Segment.  
8. Jika belum mencapai steady state, tombol Step ditekan kembali.  
9. Proses berulang hingga output stabil dan sama dengan nilai input.  

---

## Blok Diagram  
![Block Diagram Sistem](images/Blokdiagram.png)
Diagram blok dibagi menjadi tiga bagian utama:  

### 1. Blok Input  
- Clock 50 MHz  
- KEY[1] sebagai Step Trigger  
- KEY[0] sebagai Reset  
- SW[0] sebagai selector input  

### 2. Blok Processing  
- Control Unit (Edge Detection & FSM)  
- Datapath (MUX, Shift Register, Adder & Divider)  
- Konversi Data (Binary to BCD & 7-Segment Decoder)  

### 3. Blok Output  
- LEDR[9] indikator tombol  
- LEDR[0–7] indikator data input  
- HEX1 dan HEX0 sebagai display hasil filter  

---

## Finite State Machine (FSM)  
![Diagram FSM](images/fsm.png)

Sistem dirancang menggunakan **Finite State Machine (FSM)** dengan dua state utama, yaitu **STATE_IDLE** dan **STATE_SHIFT**. FSM ini berfungsi sebagai **unit kendali (control unit)** yang mengatur kapan proses pengambilan data dan pergeseran register dilakukan pada sistem.

FSM bekerja secara sinkron terhadap sinyal clock dan memiliki mekanisme reset untuk menginisialisasi sistem ke kondisi awal.

- **STATE_IDLE (Kondisi Tunggu)**  
  STATE_IDLE merupakan kondisi awal sistem setelah reset diaktifkan. Pada state ini, sistem berada dalam kondisi standby dan **tidak melakukan proses pergeseran data**.  
  FSM menonaktifkan sinyal enable ke jalur datapath sehingga nilai pada register tetap dipertahankan.  
  Jika tidak ada sinyal pemicu (trigger), FSM akan tetap berada pada STATE_IDLE.

- **STATE_SHIFT (Kondisi Proses)**  
  STATE_SHIFT merupakan kondisi aktif di mana sistem melakukan **proses pergeseran data** dan perhitungan.  
  Pada state ini, FSM mengaktifkan sinyal enable sehingga datapath dapat mengambil data baru dan menggeser isi register sesuai dengan logika sistem.

### Tabel Transisi Finite State Machine (FSM)

| State Saat Ini | Kondisi / Input | State Berikutnya | Aksi / Keterangan |
|----------------|------------------|------------------|-------------------|
| STATE_IDLE | Reset aktif | STATE_IDLE | FSM diinisialisasi ke kondisi awal |
| STATE_IDLE | Tidak ada trigger | STATE_IDLE | Sistem standby, tidak ada proses |
| STATE_IDLE | Trigger aktif (Step) | STATE_SHIFT | Memulai satu siklus pemrosesan |
| STATE_SHIFT | Satu siklus clock selesai | STATE_IDLE | Proses selesai, kembali ke standby |

FSM ini bersifat **Moore Machine**, di mana keluaran kendali (enable datapath) hanya bergantung pada **state saat ini**, bukan langsung pada input. Dengan desain dua state ini, sistem menjadi lebih sederhana, stabil, dan mudah dianalisis.

Secara keseluruhan, FSM dengan dua state ini berhasil mengatur alur kerja sistem secara terkontrol, memastikan proses hanya berjalan saat diperlukan, serta menjaga sinkronisasi antara input pengguna dan proses internal sistem.

---

# Hasil Simulasi dan Analisis

## Hasil Simulasi ModelSim

![Waveform Hasil Simulasi](images/waveform.png)

1.	Kondisi Awal (Reset):
   a.	Terlihat sinyal KEY bernilai 1110 (Bit 0 low), yang menandakan tombol Reset ditekan.
   b.	Output HEX1 bernilai 1111001 (Angka 1) dan HEX0 bernilai 1000000 (Angka 0).
   c.	Hasil: Sistem terinisialisasi dengan nilai 10.
2.	Perubahan Input (Switch):
   a.	Sinyal SW berubah dari 0 menjadi 1 (Input 90).
   b.	Sinyal LEDR langsung berubah dari ...01010 (10) menjadi ...1011010 (90).
   c.	HEX1 dan HEX0 belum berubah (masih 10). FSM menahan data sampai ada trigger.
3.	Proses Step: Setiap kali sinyal KEY berubah menjadi 1101 (Bit 1 low / Tombol Step ditekan), output berubah sesuai logika filter rata-rata:
### Tabel Hasil Perhitungan Moving Average 4-Tap

| Step Ke- | Kode HEX1 (Puluhan) | Kode HEX0 (Satuan) | Nilai Desimal | Analisis Perhitungan |
|---------|---------------------|--------------------|---------------|----------------------|
| 1 | 0110000 (Angka 3) | 1000000 (Angka 0) | **30** | (90 + 10 + 10 + 10) / 4 = **30** |
| 2 | 0010010 (Angka 5) | 1000000 (Angka 0) | **50** | (90 + 90 + 10 + 10) / 4 = **50** |
| 3 | 1111000 (Angka 7) | 1000000 (Angka 0) | **70** | (90 + 90 + 90 + 10) / 4 = **70** |
| 4 | 0010000 (Angka 9) | 1000000 (Angka 0) | **90** | (90 + 90 + 90 + 90) / 4 = **90** |

Berdasarkan analisis terhadap waveform simulasi, sistem terbukti mampu menjalankan fungsi Moving Average Filter 4-Tap dengan presisi tinggi sesuai spesifikasi perancangan. Pada fase awal, sinyal reset berhasil menginisialisasi seluruh register sehingga output HEX berada pada nilai stabil 10. Ketika terjadi perubahan input mendadak dari logika '0' (nilai 10) ke logika '1' (nilai 90) pada sinyal SW, sistem menunjukkan respons yang terbagi dua: indikator LEDR merespons seketika secara real-time, sedangkan output hasil filter (HEX) tetap bertahan pada nilai lama. Hal ini membuktikan bahwa Finite State Machine (FSM) berhasil menahan pembaruan data hingga sinyal pemicu yang valid diterima. Selanjutnya, saat tombol Step (KEY[1]) ditekan secara berurutan (terlihat pada transisi sinyal KEY ke logika low), output 7-Segment mengalami perubahan transien yang linear, yaitu bergerak dari 10 menjadi 30, 50, 70, hingga akhirnya mencapai kondisi steady state di angka 90. Respons bertahap ini memverifikasi bahwa unit aritmatika (penjumlah dan pembagi) berfungsi akurat, sekaligus memvalidasi karakteristik sistem sebagai Low Pass Filter yang efektif meredam lonjakan data input melalui mekanisme sampling sekuensia



---

## Lampiran (Kode Verilog)

### `top_system.v`
```verilog
module top_system (
    input CLOCK_50,
    input [3:0] KEY,
    input [9:0] SW,
    output [9:0] LEDR,
    output [6:0] HEX0,
    output [6:0] HEX1
);

    reg key1_prev;
    wire step_trigger;

    always @(posedge CLOCK_50) begin
        key1_prev <= KEY[1];
    end
    assign step_trigger = (key1_prev == 1'b1 && KEY[1] == 1'b0);

    reg [7:0] input_data;
    always @(*) begin
        if (SW[0] == 1'b0) input_data = 8'd10;
        else input_data = 8'd90;
    end

    localparam STATE_IDLE  = 2'd0;
    localparam STATE_SHIFT = 2'd1;

    reg [1:0] current_state, next_state;

    reg [7:0] x0, x1, x2, x3;
    wire [9:0] sum;
    wire [7:0] filter_out;

    always @(posedge CLOCK_50) begin
        if (KEY[0] == 1'b0) begin
            current_state <= STATE_IDLE;
        end else begin
            current_state <= next_state;
        end
    end

    always @(*) begin
        case (current_state)
            STATE_IDLE: begin
                if (step_trigger) 
                    next_state = STATE_SHIFT;
                else 
                    next_state = STATE_IDLE;
            end
            
            STATE_SHIFT: begin
                next_state = STATE_IDLE;
            end
            
            default: next_state = STATE_IDLE;
        endcase
    end

    always @(posedge CLOCK_50) begin
        if (KEY[0] == 1'b0) begin
            x0 <= 8'd10; 
            x1 <= 8'd10;
            x2 <= 8'd10;
            x3 <= 8'd10;
        end else begin
            if (current_state == STATE_SHIFT) begin
                x0 <= input_data; 
                x1 <= x0;          
                x2 <= x1;
                x3 <= x2;
            end
        end
    end

    assign sum = x0 + x1 + x2 + x3;
    assign filter_out = sum/4;

    reg [3:0] digit_satuan;
    reg [3:0] digit_puluhan;
    
    always @(*) begin
        digit_satuan  = filter_out % 10; 
        digit_puluhan = filter_out / 10; 
    end

    assign LEDR[7:0] = input_data; 
    
    assign LEDR[9] = ~KEY[1]; 

    seven_segment hex_low (
        .in(digit_satuan), 
        .out(HEX0)
    );
    
    seven_segment hex_high (
        .in(digit_puluhan), 
        .out(HEX1)
    );

endmodule
```

### `seven_segment.v
```verilog
- module seven_segment(
    input [3:0] in,
    output reg [6:0] out
);
 
    always @(*) begin
        case(in)
            4'h0: out = 7'b1000000; // Angka 0
            4'h1: out = 7'b1111001; // Angka 1
            4'h2: out = 7'b0100100; // Angka 2
            4'h3: out = 7'b0110000; // Angka 3
            4'h4: out = 7'b0011001; // Angka 4
            4'h5: out = 7'b0010010; // Angka 5
            4'h6: out = 7'b0000010; // Angka 6
            4'h7: out = 7'b1111000; // Angka 7
            4'h8: out = 7'b0000000; // Angka 8
            4'h9: out = 7'b0010000; // Angka 9
            4'hA: out = 7'b0001000; // A
            4'hB: out = 7'b0000011; // b
            4'hC: out = 7'b1000110; // C
            4'hD: out = 7'b0100001; // d
            4'hE: out = 7'b0000110; // E
            4'hF: out = 7'b0001110; // F
            default: out = 7'b1111111; // Mati semua
        endcase
    end
endmodule  
```
### `tb_top_system.v
```verilog
`timescale 1ns / 1ps

module tb_top_system();

    reg CLOCK_50;
    reg [3:0] KEY;
    reg [9:0] SW;
    
    wire [9:0] LEDR;
    wire [6:0] HEX0;
    wire [6:0] HEX1;

    top_system uut (
        .CLOCK_50(CLOCK_50), 
        .KEY(KEY), 
        .SW(SW), 
        .LEDR(LEDR), 
        .HEX0(HEX0), 
        .HEX1(HEX1)
    );

    initial begin
        CLOCK_50 = 0;
        forever #10 CLOCK_50 = ~CLOCK_50;
    end

    initial begin
        $display("=== SIMULASI DIMULAI ===");
        KEY = 4'b1111;
        SW = 10'b0;
        #100;

        $display("Time: %0t | Action: RESET Ditekan", $time);
        KEY[0] = 0;
        #40;
        KEY[0] = 1;
        #40;

        $display("Status Awal: Filter Output = %d", uut.filter_out);

        $display("Time: %0t | Action: Ubah SW[0] ke Atas", $time);
        SW[0] = 1;
        #40;

        press_step_button();
        $display("Step 1: Output = %d", uut.filter_out);
        
        press_step_button();
        $display("Step 2: Output = %d", uut.filter_out);
        
        press_step_button();
        $display("Step 3: Output = %d", uut.filter_out);
        
        press_step_button();
        $display("Step 4: Output = %d", uut.filter_out);
        
        press_step_button();
        $display("Step 5: Output = %d", uut.filter_out);

        $display("=== SIMULASI SELESAI ===");
        $stop;
    end

    task press_step_button;
        begin
            #20;
            KEY[1] = 0;
            #40;
            KEY[1] = 1;
            #40;
        end
    endtask

endmodule

``` 


---

## Link Video Implementasi  
https://drive.google.com/drive/folders/13rEqHvps9pyjTTlEMXvs_CQ0o9x_8JPL
