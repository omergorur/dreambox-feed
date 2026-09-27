# Dreambox One / Two paket deposu

DreamOS (opendreambox 2.6, aarch64) için hazır `.deb` paketleri.
Kutuya tek satırla eklenir, `apt` ile kurulup güncellenir.

## Amaç

DreamOS'un yazılım desteği 2017'de fiilen durdu. İmaj o günkü hâliyle
kaldı: Linux 4.9, glibc 2.25, GCC 6 dönemi libstdc++, GLib 2.50,
GStreamer 1.10, OpenSSL 1.0.2, Python 2.7. Donanım ise hâlâ güçlü:
Dreambox One/Two'daki Amlogic S922X, 4K HEVC 10-bit video çözebiliyor.
Ama üzerindeki yazılım bugünün yayınlarına, oynatıcılarına ve servislerine
yetişemiyor.

Bu deponun gayesi, desteği bitmiş bu imaja yeniden hayat vermek. Bunu
imajı bozmadan yapıyoruz: sistem dosyalarını değiştirmiyor, kutunun kendi
kütüphaneleriyle uyumlu kalıyoruz. Her paket kutuda ölçülerek ve test
edilerek hazırlanıyor, `apt` ile güncelleniyor ve kaldırılınca iz
bırakmıyor.

Şimdiye kadar yapılanlar:

- **Kodi 21.3 (Omega), donanım çözücülü:** Kodi'nin güncel sürümü,
  kutunun Mali GPU'su ve ekran sürücüsüyle çalışıyor. Video, canlı TV'nin
  kullandığı DVB donanım çözücüsünde çözülüyor: H.264, HEVC 10-bit ve
  4K50. Kutunun C++ kütüphanesi eski olduğu için Kodi, kendi
  kütüphanesini içine gömerek glibc 2.25'e göre derlendi. Tvheadend, IPTV
  Simple, Enigma2 istemcileri ve inputstream.adaptive pakete dahil.
- **Kodi 19, donanım çözücülü:** Bu yolun ilk adımı. Kodi 21 ile yan yana
  kurulabilir ama gerekli değil.
- **GStreamer HTTP kaynağı:** AJPanel stb-emu gibi Stalker portal
  eklentileriyle açılan videolar 15-30 saniyede bir başa sarıyordu.
  GStreamer 1.10'daki yeniden bağlanma hatası, aynı sürümden yamalanarak
  giderildi.
- **Transmission 4:** Tam statik derlendi; kutunun eski OpenSSL'inden ve
  kütüphanelerinden bağımsız.
- **python-coherence:** UPnP/DLNA keşfi (SSDP) için dayanıklılık yaması.
- **enigma2 eklentileri:** Günlük kullanım için iyileştirmeler:
  - EPG'de oyuncu, yönetmen ve yapım yılı bilgileri.
  - Kanal listesindeki ayraçların (marker) OpenATV'deki gibi kanal
    numarası ayırması.
  - Radyo kanallarında sabit resim yerine video.
  - Ağdan izlenen yayının kutudaki kanal değişimini izlemesi.
  - Transmission için enigma2 istemcisi.

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

### GStreamer HTTP kaynağı (yeniden bağlanma düzeltmesi)

```sh
apt-get update && apt-get install gstreamer1.0-plugins-good-souphttpsrc
systemctl restart enigma2
```

AJPanel stb-emu (Stalker portal) gibi eklentilerle açılan videoların
15-30 saniyede bir **başa sarmasını** giderir. Sunucu oynatılan videonun
bağlantısını kısa aralıklarla kesiyor; kutudaki GStreamer 1.10 yeniden
bağlanırken sunucunun tek kullanımlık yönlendirme adresini istiyor, hata
alıyor ve eklenti videoyu baştan başlatıyordu. Bu sürüm kopmada ilk adrese,
kaldığı yerden yeniden bağlanır.

Aynı GStreamer 1.10.4 sürümünden derlenmiştir; yalnızca HTTP kaynağı
değişir. Geri dönmek için orijinal paket (`1.10.4-r0.1`) yeniden kurulabilir.

### Kodi 19 (donanım çözücü destekli)

```sh
apt-get install kodi-hwdec enigma2-plugin-extensions-kodihwdec
killall -9 enigma2
```

Menüde **Kodi 19** girişi belirir. DreamOS'un kendi Kodi'si
yoktur; başka bir depodan kurulmuş bir Kodi varsa ona dokunulmaz. Bu paket
`/opt/kodi-hwdec` altına kurulur, kendi veri dizinini kullanır ve onunla
yan yana çalışabilir.

Paket kendi kendine yeter: kutunun temel feed'inde bulunmayan dört kütüphane
(`libinput`, `libevdev`, `mtdev`, `libtinyxml`) paketin içinde gelir ve
yalnızca bu uygulamaya görünür — sistem kütüphanelerine dokunulmaz, başka bir
depo eklemeniz gerekmez.

Video kutunun DVB donanım çözücüsünde çözülür: H.264, HEVC 10-bit ve HDR
dahil 4K60'a kadar. İşlemci yükü yazılım çözmeye göre onda birine iner.
**Deneysel: kare tabanlı çözücü.** `/etc/default/kodi-hwdec` içine
`export KODI_AMCODEC=1` eklenirse H.264 video kare kare çözülür ve ekrana
kare başına zamanlamayla basılır (50p'de daha akıcı; sarma ve durdurma
çalışır). HEVC (4K) bu kipte her zaman DVB çözücüsüyle oynatılır. Kapatmak
için satırı silin.

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

**Ses ile görüntü arasında sabit bir kayma olabilir** (ölçülen: 0,2–0,5 sn),
özellikle yayından kaydedilmiş TS dosyalarında ve canlı PVR'da. Kodi'nin
`Ses` ayarlarındaki gecikme (A/V offset) ile elle telafi edilebilir.

Sebebi ölçüldü: Kodi'nin Amlogic yolu normalde `/dev/video10` üzerinden
donanımın çıkardığı **her kare için** bir bildirim alır ve zaman damgasını o
karenin kendisinden okur. Bu kutuda o bildirim hiç gelmiyor — 801 kare
beslendi, `VIDIOC_DQBUF` sıfır kare döndürdü; `amlvideo` görüntü zincirine
eklendiğinde de sonuç değişmiyor. Bu yüzden gösterim zamanı örneklenerek
tahmin ediliyor ve bu tahminin çözünürlüğü kadar kayma kalıyor. Üzerinde
çalışılıyor.

Kodi düzgün kapanamazsa (çökme, elektrik kesintisi) kutu bozuk video ayarları
ile kalabilir — canlı yayın donar ya da görüntü ekranın dörtte birinde kalır.
Tek komutla toparlanır:

```sh
kodi-hwdec-restore-av
```

### Kodi 21 (donanım çözücü destekli)

```sh
apt-get install kodi21-hwdec enigma2-plugin-extensions-kodi21hwdec
killall -9 enigma2
```

Kodi'nin güncel kararlı sürümü (21.3 Omega). Menüde **Kodi 21** girişi
belirir. Tek başına kurulur; Kodi 19 paketi (`kodi-hwdec`)
**gerekmez**. İkisini birlikte kurmak isterseniz de birbirine karışmazlar:
Kodi 21 `/opt/kodi21-hwdec` altına ve kendi veri dizinine
(`/data/kodi21-hwdec`) kurulur, ayarları ve eklentileri ayrıdır.

Video kutunun DVB donanım çözücüsünde çözülür: H.264, HEVC ve MPEG2, 4K50
dahil. Tvheadend, IPTV Simple, Enigma2 (Vu+) istemcileri ve
inputstream.adaptive pakete dahildir, ayrıca kurulmaz.

Kurulum ~155 MB yer kaplar. Kök bölümde yer azsa önce `df -h /` ile
kontrol edin; indirilen paket `apt-get clean` ile silinebilir.

## Paketler

| Paket | Sürüm | Mimari | Boyut | Açıklama |
|---|---|---|---|---|
| `transmission-daemon` | 4.1.3-3 | arm64 | 23.5 MB | Transmission BitTorrent daemon (statik derlenmis) |
| `enigma2-plugin-extensions-transmission4` | 1.5 | all | 28 KB | Transmission 4.x icin enigma2 istemcisi |
| `python-coherence` | 0.8.1+git0+f39fbd2bd0-r0.0+ssdpfix2 | arm64 | 468 KB | Python UPnP framework (SSDP dayaniklilik yamalari) |
| `gstreamer1.0-plugins-good-souphttpsrc` | 1.10.4-r0.1+reconnect1 | arm64 | 23 KB | GStreamer souphttpsrc (HTTP kaynagi), yeniden baglanma yamali |
| `enigma2-plugin-extensions-eitextendeditems` | 1.4 | all | 17 KB | EPG ek bilgileri (Actors / Directors / Production Year) |
| `enigma2-plugin-extensions-markernumbering` | 1.5 | all | 20 KB | Adsiz markerlar kanal numarasi rezerve etsin (OpenATV davranisi) |
| `enigma2-plugin-extensions-radiovideo` | 1.7 | all | 3.8 MB | Radyo kanallarinda sabit resim yerine video oynat |
| `enigma2-plugin-extensions-zapfollow` | 1.3 | all | 19 KB | Agdan izlenen yayin kutudaki zap ile birlikte kanal degistirsin |
| `kodi-hwdec` | 1.0-r6 | arm64 | 30.7 MB | Donanim video cozucu destekli Kodi 19 |
| `enigma2-plugin-extensions-kodihwdec` | 1.0-r7 | all | 5 KB | Kodi 19 icin menu girisi |
| `kodi-hwdec-inputstream-adaptive` | 2.3.22 | arm64 | 1.0 MB | DASH ve HLS akis cozucusu (inputstream.adaptive) |
| `kodi-hwdec-pvr-hts` | 4.4.3 | arm64 | 261 KB | Tvheadend PVR istemcisi (pvr.hts) |
| `kodi-hwdec-pvr-iptvsimple` | 3.5.5 | arm64 | 194 KB | m3u tabanli IPTV istemcisi (pvr.iptvsimple) |
| `kodi-hwdec-pvr-vuplus` | 3.15.4 | arm64 | 355 KB | enigma2 / Vu+ PVR istemcisi (pvr.vuplus) |
| `kodi21-hwdec` | 1.0-r5 | arm64 | 57.4 MB | Donanim video cozucu destekli Kodi 21 |
| `enigma2-plugin-extensions-kodi21hwdec` | 1.0-r5 | all | 5 KB | Kodi 21 icin menu girisi |

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
