# Server SlowDNS

[Bahasa Indonesia](#bahasa-indonesia) | [English](#english)

## Instalasi Cepat (git clone)

Jalankan di VPS sebagai root. Ganti `dns.domainmu.com` dengan domain SlowDNS
Anda:

```sh
apt-get update && apt-get install -y git
rm -rf /root/slowdns
git clone https://github.com/hasyimmarjuki/slowdns-installer.git /root/slowdns
cd /root/slowdns
bash install.sh
```

Pilih **1. Install**, lalu isi pengaturan. Untuk memakai SlowDNS di UDP `53000`
sambil tetap menerima koneksi di UDP `53`, isi UDP port dengan `53000`.
Installer otomatis menambahkan redirect UDP `53` -> `53000` dan menghapusnya saat
layanan berhenti. Pastikan UDP `53` dan `53000` diizinkan pada firewall VPS dan
panel penyedia server.

Periksa layanan dan rule redirect:

```sh
systemctl status slowdns
iptables -t nat -L PREROUTING -n -v | grep slowdns-redirect
```

Hapus layanan (file tetap disimpan):

```sh
bash /root/slowdns/install.sh uninstall
```

Hapus total termasuk folder:

```sh
bash /root/slowdns/install.sh uninstall
rm -rf /root/slowdns
```

## Bahasa Indonesia

### 1. Siapkan VPS dan Berkas

Gunakan VPS Linux dengan systemd. Siapkan domain SlowDNS yang mengarah ke VPS.
Izinkan UDP `53` dan port SlowDNS yang dipilih (misalnya `53000`) pada firewall
VPS serta panel penyedia server. Untuk memakai IPv4 dan IPv6, izinkan pada
keduanya.

Masuk ke VPS sebagai root, lalu ambil berkas dari GitHub:

```sh
rm -rf /root/slowdns
git clone https://github.com/hasyimmarjuki/slowdns-installer.git /root/slowdns
cd /root/slowdns
```

Isi folder pemasangan:

```text
/root/slowdns/
  install.sh
  slowdns
  server.key
  server.pub
```

Jika tidak memakai `git clone`, unggah `install.sh` dan `slowdns` (sesuai
arsitektur VPS) ke folder tersebut.

### 2. Jalankan Installer

```sh
bash /root/slowdns/install.sh
```

Pilih **1. Install**, lalu isi pengaturan berikut. Tekan Enter untuk memakai
nilai yang ditampilkan dalam kurung siku.

| Pengaturan | Isi dengan |
| --- | --- |
| Listener IP | Tekan Enter untuk `auto`: IPv4 dan IPv6 yang tersedia. Isi satu IP VPS jika ingin memakai alamat itu saja. |
| UDP port | `53000` (dengan redirect dari `53`), atau `53` untuk langsung. |
| SlowDNS domain | Domain yang sudah Anda siapkan, misalnya `dns.domainmu.com`. |
| SSH/upstream destination | `127.0.0.1:22` jika SSH berjalan pada port `22` di VPS yang sama. |

Installer membuat konfigurasi dan pasangan key secara otomatis, lalu
menjalankan layanan `slowdns`. Konfigurasi tersimpan di `/root/slowdns/slowdns.env`,
bersama key di folder yang sama. Menu dan pesan installer memakai bahasa Inggris.
Biarkan `install.sh` di folder tersebut selama layanan digunakan.

Dengan `auto`, listener menggunakan semua IP global yang tersedia. Jika VPS
hanya memiliki IPv4 atau IPv6, keluarga alamat yang tersedia tetap dapat
digunakan. IPv6 tertentu boleh diisi sebagai `2001:db8::7` atau
`[2001:db8::7]` pada prompt IP.
Jika port sudah dipakai DNS sistem, installer berhenti tanpa mematikannya.
Anda dapat memilih satu IP VPS yang portnya masih kosong.

Pada instalasi baru, layanan berjalan sebagai root tanpa sandbox tambahan.
Pastikan hanya root yang dapat mengubah folder pemasangan dan berkas di dalamnya.

Untuk melihat bantuan perintah:

```sh
bash /root/slowdns/install.sh --help
```

`-help` dan `-h` juga tersedia. Gunakan `install` untuk langsung memasang
tanpa membuka menu.

### Redirect UDP 53 -> port SlowDNS

Jika UDP port yang dipilih bukan `53` (misalnya `53000`), installer menambahkan
aturan NAT di `PREROUTING` yang mengalihkan UDP `53` ke port tersebut. Aturan
diberi komentar `slowdns-redirect`, dipasang saat layanan start, dan dihapus saat
layanan stop. IPv4 memakai `iptables`, sedangkan IPv6 memakai `ip6tables`.
Installer tidak menambah aturan izin firewall; izinkan portnya secara manual.

Jika memilih port `53` secara langsung, installer memakai NOTRACK (raw table)
pada IP VPS yang dipilih agar trafik SlowDNS tidak memenuhi tabel pelacakan
koneksi. Aturannya juga dilepas saat layanan berhenti.

### 3. Hubungkan HTTP Custom

Lihat public key server:

```sh
cat /root/slowdns/server.pub
```

Di pengaturan SlowDNS HTTP Custom, isi IP VPS, domain, dan public key tersebut,
lalu coba hubungkan. `server.key` adalah private key dan tetap disimpan di server.
Jika memakai redirect, gunakan port `53` pada aplikasi; jika tidak, gunakan port
SlowDNS yang dipilih (misalnya `53000`).

### Memeriksa Layanan

```sh
systemctl status slowdns
journalctl -u slowdns -f
iptables -t nat -L PREROUTING -n -v | grep slowdns-redirect
```

Perintah pertama menampilkan status layanan. Perintah kedua menampilkan log
terbaru; tekan Ctrl+C untuk selesai melihat log. Perintah ketiga menampilkan
rule redirect yang aktif.

### Memasang Ulang

Sebelum mengganti `install.sh` atau binary dengan versi baru, hentikan layanan:

```sh
systemctl stop slowdns.service
```

Tunggu perintah berhasil agar aturan jaringan sebelumnya dibersihkan, lalu
unggah berkas baru tanpa mengganti `slowdns.env`, `server.key`, atau `server.pub`.

Jalankan kembali installer dan pilih **Install**. Pengaturan, key, dan unit
layanan yang sudah terpasang tetap digunakan. Layanan dimulai ulang sehingga
koneksi aktif akan terputus sementara.

### Menghapus Layanan

```sh
bash /root/slowdns/install.sh uninstall
```

Layanan dihentikan dan dihapus. Berkas konfigurasi, key, installer, dan
`slowdns` tetap tersimpan di folder pemasangan. Hapus folder secara manual jika
tidak diperlukan lagi:

```sh
rm -rf /root/slowdns
```

## English

## Quick Install (git clone)

Run on the VPS as root. Replace `dns.yourdomain.com` with your SlowDNS domain:

```sh
apt-get update && apt-get install -y git
rm -rf /root/slowdns
git clone https://github.com/hasyimmarjuki/slowdns-installer.git /root/slowdns
cd /root/slowdns
bash install.sh
```

Choose **1. Install**, then enter the settings. To run SlowDNS on UDP `53000`
while still accepting connections on UDP `53`, set the UDP port to `53000`.
The installer adds a UDP `53` -> `53000` redirect and removes it when the
service stops. Make sure UDP `53` and `53000` are allowed in the VPS firewall
and your server provider's panel.

Check the service and redirect rule:

```sh
systemctl status slowdns
iptables -t nat -L PREROUTING -n -v | grep slowdns-redirect
```

Remove the service (files are kept):

```sh
bash /root/slowdns/install.sh uninstall
```

Remove everything including the folder:

```sh
bash /root/slowdns/install.sh uninstall
rm -rf /root/slowdns
```

### 1. Prepare the VPS and Files

Use a Linux VPS with systemd. Set up your SlowDNS domain to point to the VPS.
Allow UDP `53` and the SlowDNS port you choose (for example `53000`) in the
VPS firewall and your server provider's panel. To use both IPv4 and IPv6,
allow them on both.

Log in to the VPS as root, then fetch the files from GitHub:

```sh
rm -rf /root/slowdns
git clone https://github.com/hasyimmarjuki/slowdns-installer.git /root/slowdns
cd /root/slowdns
```

Installation folder contents:

```text
/root/slowdns/
  install.sh
  slowdns
  server.key
  server.pub
```

If you do not use `git clone`, upload `install.sh` and the `slowdns` binary for
your VPS architecture to that folder.

### 2. Run the Installer

```sh
bash /root/slowdns/install.sh
```

Choose **1. Install**, then enter the settings below. Press Enter to use the
value shown in square brackets.

| Setting | Enter |
| --- | --- |
| Listener IP | Press Enter for `auto`: available IPv4 and IPv6. Enter one VPS IP to listen only on that address. |
| UDP port | `53000` (with redirect from `53`), or `53` for direct. |
| SlowDNS domain | The domain you set up, such as `dns.yourdomain.com`. |
| SSH/upstream destination | `127.0.0.1:22` if SSH runs on port `22` on the same VPS. |

The installer creates the configuration and key pair, then starts the
`slowdns` service. The configuration is saved in `/root/slowdns/slowdns.env`,
with the keys in the same folder. Installer menus and messages are in English.
Keep `install.sh` in that folder while the service is in use.

With `auto`, the listener uses every available global IP. If the VPS only has
IPv4 or IPv6, it can still use the available address family. You can enter a
specific IPv6 address as `2001:db8::7` or `[2001:db8::7]` at the IP prompt.
If system DNS is already using the port, the installer stops without stopping it.
You can choose one VPS IP where the port is free instead.

New installations run the service as root without an added sandbox.
Keep the installation folder and its files writable only by root.

To view command help:

```sh
bash /root/slowdns/install.sh --help
```

`-help` and `-h` also work. Use `install` to install directly without
opening the menu.

### UDP 53 -> SlowDNS port redirect

If the chosen UDP port is not `53` (for example `53000`), the installer adds a
NAT `PREROUTING` rule that redirects UDP `53` to that port. The rule is tagged
with the comment `slowdns-redirect`, added when the service starts, and removed
when it stops. IPv4 uses `iptables`, and IPv6 uses `ip6tables`.
The installer does not add firewall allow rules; allow the ports yourself.

If you choose port `53` directly, the installer uses NOTRACK (raw table) on the
selected VPS IPs so SlowDNS traffic stays out of the connection tracking table.
Those rules are also removed when the service stops.

### 3. Connect HTTP Custom

View the server's public key:

```sh
cat /root/slowdns/server.pub
```

In HTTP Custom's SlowDNS settings, enter the VPS IP address, domain, and
public key, then try connecting. `server.key` is the private key and stays
on the server. If you use the redirect, use port `53` in the app; otherwise use
the SlowDNS port you chose (for example `53000`).

### Check the Service

```sh
systemctl status slowdns
journalctl -u slowdns -f
iptables -t nat -L PREROUTING -n -v | grep slowdns-redirect
```

The first command shows the service status. The second shows the latest logs;
press Ctrl+C when you finish viewing them. The third shows the active redirect
rules.

### Reinstall

Before replacing `install.sh` or the binary with a new version, stop the service:

```sh
systemctl stop slowdns.service
```

Wait for the command to succeed so the previous network rules are removed, then
upload the new files without replacing `slowdns.env`, `server.key`, or `server.pub`.

Run the installer again and choose **Install**. Your saved settings, keys, and
existing service unit are reused. The service restarts, briefly disconnecting
active connections.

### Uninstall

```sh
bash /root/slowdns/install.sh uninstall
```

The service is stopped and removed. The configuration, keys, installer, and
`slowdns` remain in the installation folder. Remove the folder manually if it is
no longer needed:

```sh
rm -rf /root/slowdns
```
