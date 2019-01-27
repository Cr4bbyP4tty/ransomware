# Ransomware

[! [Status Pembuatan] (https://travis-ci.org/mauri870/ransomware.svg?branch=master)] (https://travis-ci.org/mauri870/ransomware)

> Catatan: Proyek ini murni bersifat akademis, gunakan dengan risiko Anda sendiri. Saya tidak menganjurkan dengan cara apa pun penggunaan perangkat lunak ini secara ilegal atau untuk menyerang target tanpa izin mereka sebelumnya

** Maksudnya di sini adalah untuk menyebarluaskan dan mengajarkan lebih banyak tentang keamanan di dunia nyata. Ingat, keamanan selalu merupakan pedang bermata dua **

### Apa itu Ransomware?
Ransomware adalah jenis malware yang mencegah atau membatasi pengguna mengakses sistem mereka, baik dengan mengunci layar sistem atau dengan mengunci file pengguna kecuali tebusan dibayarkan. Lebih banyak keluarga ransomware modern, secara kolektif dikategorikan sebagai crypto-ransomware, mengenkripsi tipe file tertentu pada sistem yang terinfeksi dan memaksa pengguna untuk membayar uang tebusan melalui metode pembayaran online tertentu untuk mendapatkan kunci dekripsi.

### Ringkasan proyek
Proyek ini bertujuan untuk membangun crypto-ransomware yang hampir berfungsi untuk tujuan pendidikan, ditulis dalam Go. Pada dasarnya, itu akan mengenkripsi file Anda di latar belakang menggunakan AES-256-CTR, algoritma enkripsi yang kuat, menggunakan RSA-4096 untuk mengamankan pertukaran kunci dengan server. Ya, Cryptolocker menyukai malware.

Ini terdiri dari dua bagian utama, server dan malware itu sendiri.

Server bertanggung jawab untuk menyimpan Id dan kunci enkripsi masing-masing dan mungkin bertindak sebagai server Command and Control dalam waktu dekat.

Malware mengenkripsi dengan kunci publik RSA-4096 Anda muatan apa pun sebelum mengirim lalu ke server. Pendekatan ini dengan transportasi https bersama-sama membuat keamanan dan otentikasi hampir tidak dapat dipecahkan (secara teori)

### Tugas proyek

- [x] Jalankan di Latar Belakang (atau tidak)
- [x] Enkripsi file menggunakan AES-256-CTR (Mode Penghitung) dengan IV acak untuk setiap file
- [x] Tanpa tanda tangan virus (saat ini)
- [x] Gunakan RSA-4096 untuk mengamankan keaslian
- [x] HTTPS dan HTTP \ 2 Transport secara default
- [x] Alirkan enkripsi untuk menghindari memuat seluruh file ke dalam memori
- [x] Berjalan semua drive secara default, termasuk lokasi usb dan jaringan
- [] Kunci entri registri dengan hash digest (mungkin SHA-256) untuk mengidentifikasi korban yang terinfeksi
- [] Tor atau pendekatan lain untuk menyembunyikan koneksi dengan C&C [lihat edisi 3] (https://github.com/mauri870/ransomware/issues/3)
- [x] Gambar Docker untuk dikompilasi

### Building the binaries

> DON'T RUN ransomware.exe IN YOUR PERSONAL MACHINE, EXECUTE ONLY IN A TEST ENVIRONMENT!

#### Docker

```
go get -v github.com/mauri870/ransomware
cd $GOPATH/src/github.com/mauri870/ransomware
# You can compile the server for windows using env GOOS=windows make instead of make
sh build-docker.sh make
```

Done! The binaries live on the bin folder

#### Local

You need Go at least 1.7 with the `$GOPATH/bin` in your $PATH

```
go get -v github.com/Cr4bbyP4tty/ransomware
go get -v github.com/akavel/rsrc
cd $GOPATH/src/github.com/Cr4bbyP4tty/ransomware
```

Build the project require a lot of steps, like the RSA key generation, build three binaries, embed manifest files, so, let's leave `make` do your job
```
make
```
If you like build the server for windows from a unix machine, run `env GOOS=windows make`.

> DON'T RUN ransomware.exe IN YOUR PERSONAL MACHINE, EXECUTE ONLY IN A TEST ENVIRONMENT!

## Usage and How it Works

The malware will run in background. You can see what is going on by simply remove the `-ldflags="-H windowsgui"` from the binaries section on Makefile before build

By default, the server will listen on `https://localhost:8080`. The client will use this host as the default url too.

You can put the server on any domain and start it. Simply overwrite the `SERVER_URL` constant on `client/main.go` before build and the malware will try to connect with this url instead

After build, a binary called `ransomware.exe`, `server`/`server.exe` and `unlocker.exe` will be generated on the bin folder. The execution of `ransomware.exe` and `unlocker.exe` (even if it is compiled for linux/darwin) is locked to windows machines only.

Feel free to edit the parameters across the files for testing.
Put the binaries on a correct windows test environment and start the server.
It will wait for the malware contact and persist the id/encryption keys

When double click on `ransomware.exe` binary it will run on background, walking interesting directories and encrypting all files that match the interesting file extensions using AES-256-CTR and a random IV, recreating then with encrypted content and a custom extension(.encrypted by default) and create a READ_TO_DECRYPT.html file on desktop

In theory, for decrypt your files you need send an amount of BTC to the attacker's wallet, followed by a contact sending your ID(located on the file created on desktop). If your payment was confirmed, the attacker possibly will return your encryption key and the `unlocker.exe` and you can use then to recover your files. This exchange can be accomplished in several ways and is not been implemented yet.

Let's suppose you get your encryption key back, you can retrieve it pointing to the following url:

```
curl -k https://localhost:8080/api/keys/:id
```
Where `:id` is your identification stored on the file on desktop. After, run the `unlocker.exe` by double click and follow the instructions.

And that's it, got your files back :smile:

## Server endpoints

The server has only two endpoints at the moment

`POST api/keys/add` - Used by the malware to persist new keys. Some verifications are made, like the verification of the RSA autenticity. Returns 204 (empty content) in case of success or a json error.

`GET api/keys/:id` - Id is a 32 characters parameter, representing an Id already persisted. Returns a json containing the encryption key or a json error

## The end

As you can see, building a functional ransomware, with some of the best existing algorithms is not dificult, anyone with programming and security skills can build that.
