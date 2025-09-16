# Build Your Own Python Package

*Basic Python for Cybersecurity*

*© 2025 Yogi Agnia Dwi Saputro dan PT. Cyberkarta Tugu Teknologi.*

Dilarang keras memperbanyak, menyalin, mendistribusikan, atau menggunakan sebagian atau seluruh isi karya ini dalam bentuk apapun tanpa izin tertulis dari pemegang hak cipta.

------------------------------------

## Mengapa Perlu Publish?
- Membangun portofolio
- Membangun kredibilitas

## Publish di Mana?
- Github (https://github.com) --> public repo
- pypi.org (https://pypi.org) --> bisa dipakai orang dengan `pip install`

## Prasyarat
- Punya email
- Punya akun Github dan pypi.org. Kalau belum punya, setup dan daftar dulu:
  - github: https://github.com/signup
  - pypi.org: https://pypi.org/account/register/
- *Note* -- website modern biasanya meminta user untuk aktifkan 2FA/MFA.
  - Persiapkan nomor handphone atau authenticator app.
  - Kamu akan diberi recovery codes. Simpan recovery codes di tempat yang aman.
- Buat API Token Github (opsional)
  - Masuk ke menu Settings → Developer settings → Personal access tokens
  - Pilih Tokens (classic) atau Fine-grained tokens (aspek security lebih baik).
  - Klik **Generate new token**, pilih scope (misalnya repo, workflow).
  - Salin token yang muncul (hanya ditampilkan sekali).
- Buat API Token pypi.org
  - Masuk ke Account settings → API tokens.
  - Klik **Add API token**, beri nama, lalu pilih scope.
  - Salin token yang muncul (hanya ditampilkan sekali).

## Praktik
- Contoh: `is-probably-hash` --> cek apakah suatu string kemungkinan adalah hash berdasarkan panjang string
- Struktur project
```
is-probably-hash/
├── is-probably-hash/              # package folder
│   ├── __init__.py
│   └── main.py
├── README.md
├── LICENSE
├── pyproject.toml
└── setup.cfg
```

### Mulai Koding
- `__init__.py` untuk pilih bagian library mana yang dipakai
```python
# is-probably-hash/__init__.py
from .main import hello, is_potential_hash
```

- `main.py` untuk logic utama library
```python
# is-probably-hash/main.py

import re

# Known hash lengths by algorithm
HASH_LENGTHS = {
    "MD5": 32,
    "SHA1": 40,
    "SHA256": 64,
    "SHA512": 128
}

HEX_PATTERN = re.compile(r"^[a-fA-F0-9]+$")


def is_probably_hash(s: str):
    """
    Check if the string is a potential hash based on length and allowed characters.
    Returns: (bool, algorithm or None)
    """
    s = s.strip()

    if HEX_PATTERN.fullmatch(s):
        for algo, length in HASH_LENGTHS.items():
            if len(s) == length:
                return True, algo
        return True, None
    return False, None


def hello():
    print("mytool hash detector is ready! Use is_probably_hash() to check strings.")


if __name__ == "__main__":
    test_strings = [
        "5d41402abc4b2a76b9719d911017c592",  # MD5
        "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3",  # SHA1
        "notahash"
    ]
    for s in test_strings:
        result, algo = is_probably_hash(s)
        if result:
            print(f"{s} looks like a hash ({algo or 'Unknown'})")
        else:
            print(f"{s} is NOT a hash")

```

- `pyproject.toml` untuk setup build library
```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

- `README.md` untuk informasi bagi pengguna library. 
```markdown
# Judul Library
...

## Info Install
...

## Info Bug & Kontribusi
...
```

- `setup.cfg` untuk informasi yang dibutuhkan ekosistem Python
```cfg
[metadata]
name = is_probably_hash
version = 0.1.0
author = Yogi Saputro
author_email = yogisaputro@outlook.com
description = My awesome CLI tool
long_description = file: README.md
long_description_content_type = text/markdown
url = https://github.com/yogski/is_probably_hash
license = MIT

[options]
packages = find:
python_requires = >=3.7

[options.entry_points]
console_scripts =
    is_probably_hash = is_probably_hash.main:hello
```

### Publikasi

#### Github
Publikasi di Github bisa dengan 2 cara: 
1. Upload secara manual di interface web Github
2. Buat repo sync local-remote Github dengan API key atau SSH key

#### pypi.org
Build dan distribusi dengan 3 langkah
```bash
pip install build twine

python -m build

python -m twine upload dist/*
```
*Note*: akan ada prompt untuk masukkan API token. Paste API token yang sudah kamu buat. 

## Referensi
- Dokumentasi Token API Github: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens?utm_source=chatgpt.com
- Dokumentasi Token API pypi.org: https://pypi.org/help/?#apitoken