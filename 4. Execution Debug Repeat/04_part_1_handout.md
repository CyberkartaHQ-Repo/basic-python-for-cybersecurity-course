# Execute, Debug Repeat

## *Part* 1: Eksekusi Program
Di bagian ini fokusnya adalah membuat program secara spesifik.

Sebaiknya siap-siap untuk berpikir, mengetik, menerima error, dan melakukan perbaikan.

*Copy-paste* tidak dilarang, namun sangat dianjurkan bagi teman-teman pembelajar untuk mengetik ulang code di bagian ini. Mengetik dan membiasakan dengan *syntax* adalah bagian penting proses belajar.

Menggunakan generative AI untuk mengetahui fungsi code dipersilakan. Harap diperhatikan bahwa AI tersebut memberikan respon umum dan memberikan beberapa sugesti yang mungkin tidak sesuai konteks. 

Code yang ada di bawah bisa jadi mengandung kesalahan, dan itu juga bagian dari desain *course* ini. Seiring bertambahnya materi yang dipelajari, teman-teman pembelajar akan memahami kesalahan tersebut dan memperbaiki code yang ada.

## Reminder: DRIVE Framework

- **D**efine: tentukan input-output-proses program
- **R**efine: jabarkan *step-by-step* logika proses 
- **I**mplement/**I**terate: tulis code sampai program jalan tanpa error
- **V**alidate: cek kembali apakah input-output sudah benar untuk berbagai kasus
- **E**nhance: program sudah oke? lanjut. Belum oke? perbaiki lagi.

## Praktik: *Password Strength Checker*

Membuat program untuk cek apakah password kuat/lemah. Program ini masih di tahap pertama dan akan diperbaiki lagi sepanjang *course* ini.

```python
# password strength checker

# -------- D: Define ----------
# input: password input (string)
# process: analyze password to best practice an score its strength
# output: scoring from 1 - weakest to 10 - strongest (int)

# -------- R: Refine ----------
import string

password_input = "password"
score = 0

# password length - 3 points
# more refine: length 0-3: 0 point, 4-7: 1 point, 8-12: 2 points, >12 : 3 points
length_password = len(password_input)
score += int(length_password / 4)

# password has uppercase - 1 point
has_uppercase = any(char.isupper() for char in password_input) # check if each char is uppercase
score += int(has_uppercase) # add to score true = 1, false = 0

# password has lowercase - 1 point
has_lowercase = any(char.islower() for char in password_input)
score += int(has_lowercase)

# password has number - 1 point
has_digits = any(char.isdigit() for char in password_input)
score += int(has_digits)

# password has special character - 2 point
# char is special for each letters checked
has_special_chars = any(char in string.punctuation for char in password_input)
score += int(has_special_chars)

# password has no common words - 1 point
common_words = ["pass", "password", "admin", "user", "me"]
not_common_words = not any(word in password_input.lower() for word in common_words)
score += int(not_common_words)

# password is not in list of leaked passwords - 1 point
leaked_passwords = ["123456", "pass", "password", "admin", "qwerty", "yahoo"]
not_leaked_words = not any(word in password_input for word in leaked_passwords)

print("password input:")
print(password_input)
print("password score out of 10:")
print(score)

# password: 3 --> length 8 (2 points) + lowercase (1 point)
```

## *Extra Challenge*

Berdasarkan score password, tambahkan emoji untuk memberikan output yang lebih ekspresif.
- 1–3: 😢
- 4–6: 😐
- 7–10: 😎

## Referensi
- Cek kekuatan password secara online: https://www.security.org/how-secure-is-my-password/
- https://www.petanikode.com/python-list/
- https://belajarpython.id/fungsi-bawaan/
- https://www.petanikode.com/python-casting/