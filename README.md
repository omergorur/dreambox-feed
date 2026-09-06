# Dreambox One / Two paket deposu

DreamOS (opendreambox 2.6, aarch64) için hazır `.deb` paketleri.
Kutuya tek satırla eklenir, `apt` ile kurulup güncellenir.

## Kurulum

Kutuya SSH ile bağlanın ve depoyu ekleyin:

```sh
echo "deb [trusted=yes] https://omergorur.github.io/dreambox-feed/ ./" > /etc/apt/sources.list.d/dreambox-feed.list
apt-get update
```

Sonra istediğiniz paketi kurun:

```sh
apt-get install transmission-daemon enigma2-plugin-extensions-transmission4
killall -9 enigma2
```

Eklentiler `killall -9 enigma2` sonrası devreye girer; `transmission-daemon`
ve `python-coherence` için buna gerek yoktur (ilki systemd servisini kendi
başlatır, ikincisi enigma2 yeniden başlayınca etkinleşir).

### Neden iki paket birden?

`enigma2-plugin-extensions-transmission4` daemon'ı `Recommends` olarak
işaretler, `Depends` olarak değil — çünkü eklenti ağdaki **başka bir
makinedeki** Transmission'a da bağlanabilir (NAS, PC). Çoğu kutuda
`APT::Install-Recommends "1"` olduğu için daemon kendiliğinden gelir, ama
bu ayar kapalıysa gelmez ve eklenti "Bağlantı yok" der. Yukarıdaki komut
ikisini birden istediği için her koşulda çalışır.

Kutunuzdaki ayarı görmek için: `apt-config dump | grep Recommends`

Daemon zaten ağdaki başka bir makinedeyse yalnızca eklentiyi kurun ve
adresini **Menü → Eklentiler → Transmission → Menü → Ayarlar**'dan girin.

## Paketler

| Paket | Sürüm | Mimari | Boyut | Açıklama |
|---|---|---|---|---|
| `transmission-daemon` | 4.1.3-3 | arm64 | 23.5 MB | Transmission BitTorrent daemon (statik derlenmis) |
| `enigma2-plugin-extensions-transmission4` | 1.4 | all | 28 KB | Transmission 4.x icin enigma2 istemcisi |
| `python-coherence` | 0.8.1+git0+f39fbd2bd0-r0.0+ssdpfix2 | arm64 | 468 KB | Python UPnP framework (SSDP dayaniklilik yamalari) |
| `enigma2-plugin-extensions-eitextendeditems` | 1.3 | all | 17 KB | EPG ek bilgileri (Actors / Directors / Production Year) |
| `enigma2-plugin-extensions-markernumbering` | 1.4 | all | 19 KB | Adsiz markerlar kanal numarasi rezerve etsin (OpenATV davranisi) |
| `enigma2-plugin-extensions-radiovideo` | 1.6 | all | 3.8 MB | Radyo kanallarinda sabit resim yerine video oynat |
| `enigma2-plugin-extensions-zapfollow` | 1.2 | all | 19 KB | Agdan izlenen yayin kutudaki zap ile birlikte kanal degistirsin |

## Notlar

- Paketler **aarch64 / arm64** içindir (Dreambox One, Dreambox Two).
- `transmission-daemon` **tam statik** derlenmiştir: kutunun OpenSSL 1.0.2,
  libevent 2.0 ve libstdc++ 6.0.22 sürümleri onu ilgilendirmez. Mevcut
  `settings.json` ve torrent dosyalarınıza dokunmaz.
- `python-coherence` upstream paketin yamalı sürümüdür; sürüm numarası
  `+ssdpfix2` eki taşır. Feed'den gerçek bir güncelleme gelirse (`-r0.1`)
  apt onu tercih eder, yani depo sizi upstream'e kilitlemez.
- Kaldırmak için: `apt-get remove <paket-adı>`
- Depoyu kaldırmak için: `rm /etc/apt/sources.list.d/dreambox-feed.list`
