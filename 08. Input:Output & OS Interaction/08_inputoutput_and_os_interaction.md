# Input/Output and OS Interaction

*Basic Python for Cybersecurity*

*© 2025 Yogi Agnia Dwi Saputro dan PT. Cyberkarta Tugu Teknologi.*

Dilarang keras memperbanyak, menyalin, mendistribusikan, atau menggunakan sebagian atau seluruh isi karya ini dalam bentuk apapun tanpa izin tertulis dari pemegang hak cipta.

------------------------------------

## Input/Output

Secara umum, program menyediakan berbagai versi input dan output sesuai dengan *use case* dan pihak yang berinteraksi (manusia/sistem lain).

### Input
| Metode                | User            | Deskripsi                       | Implementasi                          |
| --------------------- | --------------- | --------------------------------- | -------------------------------- |
| **Input interaktif**  | Manusia         | User memberikan input saat program berjalan | `input("Enter your name: ")`     |
| **Command-line args** | Manusia/sistem  | Parameter yang diteruskan ketika program akan dijalankan    | `sys.argv[1]` using `import sys` |
| **File**              | Manusia/sistem  | Membaca file eksternal sebagai sumber input    | `open("log.txt").read()`         |
| **Variabel env**      | Sistem          | Variabel untuk config dan key/secret       | `os.getenv("API_KEY")`           |

### Output
| Metode                | User            | Deskripsi                       | Implementasi                          |
| --------------------- | --------------- | --------------------------------- | -------------------------------- |
| **Console**  | Manusia/sistem         | Program menuliskan informasi di console ketika sedang berjalan. Paling mudah diimplementasikan. *Note*: output console akan hilang jika program mati/restart atau user keluar dari terminal/shell. | `print("[INFO] Database connected!")`     |
| **Output File** | Manusia/sistem  | Program menuliskan data ke dalam file yang sifatnya persisten.    | `open("20250430101948_monitoring.log", "w").write(data)` |
| **Return value**              | Sistem  | Program mengembalikan value kepada pihak yang menjalankannya.  | `return result`         |

## Interaksi dengan OS (Operating System)

OS pada dasarnya adalah "lingkungan" dari aplikasi. Analoginya: jika program/aplikasi adalah toko, maka OS adalah mall. Toko bersinergi dengan mall untuk memberikan pengalaman terbaik bagi pengguna. Untuk itu, ada standar interaksi yang dapat dilakukan.

Dalam konteks program-OS, program umumnya berurusan dengan hal terkait:
- navigasi path
- manajemen resource: CPU, memory, dll
- manajemen akses
- manajemen process/subprocess

Contoh instruksi terkait OS di Python:
| Perintah                  | Syntax Python                                    |
| --------------------- | ------------------------------------------ |
| List files            | `os.listdir(".")`                          |
| Jalankan perintah (seolah-olah user ketik di shell)   | `os.system("ls -l")` or `subprocess.run()` |
| Cek file  | `os.path.exists("config.txt")`             |
| Cek folder saat ini | `os.getcwd()`                              |
| Pindah folder   | `os.chdir("/home/user")`                   |

Tentunya ada lebih banyak instruksi yang tersedia. Coba eksplorasi dan jalankan di komputer kamu.

## Interaksi Antarprogram

Program Python dapat berinteraksi dengan program Python lain melalui beberapa mekanisme.
| Method                 | Use Case                     | Example                                  |
| ---------------------- | ---------------------------- | ---------------------------------------- |
| **Import as module**   | Code digunakan ulang                  | `import utils`                           |
| **Subprocess**         | Satu program menjalankan program lain, peran sebagai eksekutor mewakili user      | `subprocess.run(["python3", "tool.py"])` |
| **Socket/network**     | Dua program memiliki interface (HTTP, websocket, dll) dan bertukar data           | `socket.socket()` (advanced)             |
| **Inter-process file** | Berbagi via files atau logs | Read/write file yang sama                |

Manfaat interaksi antarprogram
- Setiap developer dapat berkontribusi di bagian tertentu
- Mendorong kolaborasi
- Mendorong spesialisasi program, lebih mudah dikelola
- Mendorong desain modular dengan banyak pilihan, misal kita bisa memilih library untuk fungsi HTTP request

## Praktik: Log Analysis Pipeline (Extraction + Reporting)

Kali ini kamu akan membuat pipeline analisis log. Prosesnya dijalankan oleh beberapa program yang berkesinambungan.

Untuk kasus ini, pipeline terdiri dari dua program:
1. `Extractor` --> mengambil data sensitif dari log, persis dengan Chapter 07, perbedaannya adalah: file log terpisah, return value, dan tambahan instruksi untuk melanjutkan ke proses berikutnya
2. `Reporter` --> menerima input dari `Extractor`, lalu mengirimkan datanya ke service lain via HTTP request

- File `extractor.py`
```python
# extractor.py
# For educational purposes only

import sys
import json
import subprocess

SENSITIVE_KEYS = ["username", "password", "api_key", "token", "secret"]

def extract_sensitive_data(filepath):
    findings = {}

    try:
        with open(filepath, "r") as file:
            lines = file.readlines()

        for key in SENSITIVE_KEYS:
            findings[key] = []

            for line in lines:
                if key in line.lower():
                    findings[key].append(line.strip())

    except Exception as e:
        print(f"[!] Error reading log file: {e}")
        sys.exit(1)

    return findings


def send_to_reporter(data_dict):
    # Convert ke JSON string
    json_data = json.dumps(data_dict)

    # Panggil reporter.py dengan input JSON 
    subprocess.run(["python3", "reporter.py", json_data])


if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python3 extractor.py <logfile>")
        sys.exit(1)

    log_path = sys.argv[1]
    print(f"[*] Analyzing {log_path}...")

    sensitive_data = extract_sensitive_data(log_path)
    print(f"[+] Found sensitive items: {sensitive_data}")

    send_to_reporter(sensitive_data)
```

- File `reporter.py`
```python
# reporter.py
# For educational purposes only

import sys
import json
import requests

MOCK_ENDPOINT = "https://httpbin.org/post"  # bisa diganti ke endpoint lain

def send_report(data):
    try:
        response = requests.post(MOCK_ENDPOINT, json=data)
        print(f"[+] Sent report. Server responded with {response.status_code}")
        print(response.json())
    except Exception as e:
        print(f"[!] Failed to send report: {e}")


if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python3 reporter.py '<json_data>'")
        sys.exit(1)

    try:
        sensitive_dict = json.loads(sys.argv[1])
        send_report(sensitive_dict)
    except json.JSONDecodeError:
        print("[!] Invalid JSON input.")
```

## *Extra Challenge*

- Asumsikan ada file log yang di-generate setiap jam, buat agar pipeline tersebut selalu jalan, mengecek apakah ada file log terbaru (bisa auto deteksi file lama di-skip), lalu memprosesnya.
- Tantangan mandiri: buat program Python yang bisa mengkloning dirinya sendiri, alias menjalankan subprocess untuk memanggil dirinya sendiri :)

## Referensi
- Modul `os` Python: https://www.w3schools.com/python/module_os.asp
- Modul `subprocess` Python: https://www.geeksforgeeks.org/python/python-subprocess-module/
- Fork bomb: https://id.wikipedia.org/wiki/Fork_bomb