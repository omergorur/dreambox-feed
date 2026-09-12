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

### Donanım çözücü destekli Kodi

```sh
apt-get install kodi-hwdec enigma2-plugin-extensions-kodihwdec
killall -9 enigma2
```

Menüde **Kodi (donanım çözücü)** girişi belirir. Kutunun kendi *Kodi
MediaCenter* girişi yerinde kalır; ikisi yan yana çalışır, istediğinizi
kullanırsınız.

Paket kendi kendine yeter: kutunun temel feed'inde bulunmayan dört kütüphane
(`libinput`, `libevdev`, `mtdev`, `libtinyxml`) paketin içinde gelir ve
yalnızca bu uygulamaya görünür — sistem kütüphanelerine dokunulmaz, başka bir
depo eklemeniz gerekmez.

Video kutunun DVB donanım çözücüsünde çözülür: H.264, HEVC 10-bit ve HDR
dahil 4K60'a kadar. İşlemci yükü yazılım çözmeye göre onda birine iner.
Kodi'nin kendi donanım yolları (v4l2m2m, amcodec) bu çekirdekte çalışmaz —
sürücü çözülen kareyi userspace'e hiç vermez.

Veri dizini varsayılan olarak `/data/kodi-hwdec`. Küçük resim önbelleği
büyüyebilir; sabit diskiniz varsa `/etc/default/kodi-hwdec` içinden
değiştirin:

```sh
KODI_HWDEC_DATA=/media/hdd/kodi-hwdec
```

#### Eklentiler

Kodi'nin ikili eklentileri ayrı paketlerdir; yalnızca kullanacaklarınızı kurun:

```sh
apt-get install kodi-hwdec-inputstream-adaptive   # DASH / HLS akışları
apt-get install kodi-hwdec-pvr-hts                # Tvheadend
apt-get install kodi-hwdec-pvr-iptvsimple         # m3u + XMLTV
apt-get install kodi-hwdec-pvr-vuplus             # enigma2 / Vu+ stream ucu
```

Bunlar `kodi-hwdec` ağacının içine kurulur ve kaynağından derlenmiştir;
`libkodiplatform` gibi ek bir depo bağımlılığı getirmezler.

`inputstream.adaptive` Widevine DRM korumalı servislerde (Netflix, Disney+
vb.) **çalışmaz** — aarch64 için Widevine CDM yoktur. Şifresiz DASH/HLS
akışlarında sorunsuzdur.

#### Bilinen kusur

**TV'den kaydedilip MKV'ye çevrilmiş yayın dosyalarında** oynatmanın ilk
~10 saniyesi takılır, sonra kendiliğinden düzelir. İleri sarmak da düzeltir.
Diğer her şey (web indirmeleri, 4K HDR dahil) tam kare hızında, kare
düşürmeden oynar.

Sebebi ölçüldü: kareler çözme sırasında beslenir ama her birine gösterim
zaman damgası iliştirilir. Yayın kodlayıcıları derin B-kare piramidi
kullandığı için bu damgalar sırasız gelir (ölçülen: `0.00, 0.32, 0.16,
0.08…`) ve görüntü katmanı kareleri "geç kalmış" sayıp atar. Çözümü
üzerinde çalışılıyor; düzelince yeni sürüm yayınlanacak.

Kodi düzgün kapanamazsa (çökme, elektrik kesintisi) kutu bozuk video ayarları
ile kalabilir — canlı yayın donar ya da görüntü ekranın dörtte birinde kalır.
Tek komutla toparlanır:

```sh
kodi-hwdec-restore-av
```

## Paketler

| Paket | Sürüm | Mimari | Boyut | Açıklama |
|---|---|---|---|---|
| `transmission-daemon` | 4.1.3-3 | arm64 | 23.5 MB | Transmission BitTorrent daemon (statik derlenmis) |
| `enigma2-plugin-extensions-transmission4` | 1.5 | all | 28 KB | Transmission 4.x icin enigma2 istemcisi |
| `python-coherence` | 0.8.1+git0+f39fbd2bd0-r0.0+ssdpfix2 | arm64 | 468 KB | Python UPnP framework (SSDP dayaniklilik yamalari) |
| `enigma2-plugin-extensions-eitextendeditems` | 1.4 | all | 17 KB | EPG ek bilgileri (Actors / Directors / Production Year) |
| `enigma2-plugin-extensions-markernumbering` | 1.5 | all | 20 KB | Adsiz markerlar kanal numarasi rezerve etsin (OpenATV davranisi) |
| `enigma2-plugin-extensions-radiovideo` | 1.7 | all | 3.8 MB | Radyo kanallarinda sabit resim yerine video oynat |
| `enigma2-plugin-extensions-zapfollow` | 1.3 | all | 19 KB | Agdan izlenen yayin kutudaki zap ile birlikte kanal degistirsin |
| `kodi-hwdec` | 1.0 | arm64 | 30.7 MB | Donanim video cozucu destekli Kodi 19 |
| `enigma2-plugin-extensions-kodihwdec` | 1.0 | all | 5 KB | Donanim cozucu destekli Kodi icin menu girisi |
| `kodi-hwdec-inputstream-adaptive` | 2.3.22 | arm64 | 1.0 MB | DASH ve HLS akis cozucusu (inputstream.adaptive) |
| `kodi-hwdec-pvr-hts` | 4.4.3 | arm64 | 261 KB | Tvheadend PVR istemcisi (pvr.hts) |
| `kodi-hwdec-pvr-iptvsimple` | 3.5.5 | arm64 | 194 KB | m3u tabanli IPTV istemcisi (pvr.iptvsimple) |
| `kodi-hwdec-pvr-vuplus` | 3.15.4 | arm64 | 355 KB | enigma2 / Vu+ PVR istemcisi (pvr.vuplus) |

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
