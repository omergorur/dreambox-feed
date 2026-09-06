# Dreambox One / Two paket deposu

DreamOS (opendreambox 2.6, aarch64) icin hazir `.deb` paketleri.
Kutuda tek satirla kurulur, `apt` ile guncellenir.

## Kurulum

Kutuya SSH ile baglanin ve depoyu ekleyin:

```sh
echo "deb [trusted=yes] https://omergorur.github.io/dreambox-feed/ ./" > /etc/apt/sources.list.d/dreambox-feed.list
apt-get update
```

Sonra istediginiz paketi kurun:

```sh
apt-get install enigma2-plugin-extensions-transmission4
killall -9 enigma2
```

## Paketler

| Paket | Surum | Mimari | Boyut | Aciklama |
|---|---|---|---|---|
| `transmission-daemon` | 4.1.3-2 | arm64 | 23.5 MB | Transmission BitTorrent daemon (statik derlenmis) |
| `enigma2-plugin-extensions-transmission4` | 1.3 | all | 0.0 MB | Transmission 4.x icin enigma2 istemcisi |
| `python-coherence` | 0.8.1+git0+f39fbd2bd0-r0.0+ssdpfix2 | arm64 | 0.5 MB | Python UPnP framework (SSDP dayaniklilik yamalari) |
| `enigma2-plugin-extensions-eitextendeditems` | 1.2 | all | 0.0 MB | EPG ek bilgileri (Actors / Directors / Production Year) |
| `enigma2-plugin-extensions-markernumbering` | 1.3r-2 | all | 0.0 MB | Adsiz markerlar kanal numarasi rezerve etsin (OpenATV davran |
| `enigma2-plugin-extensions-radiovideo` | 1.5 | all | 3.8 MB | Radyo kanallarinda sabit resim yerine video oynat |
| `enigma2-plugin-extensions-zapfollow` | 1.1r-3 | all | 0.0 MB | Agdan izlenen yayin kutudaki zap ile birlikte kanal degistir |

## Notlar

- Paketler **aarch64 / arm64** icindir (Dreambox One, Dreambox Two).
- `python-coherence` upstream paketin yamali surumudur: surum numarasi
  `+ssdpfix2` eki tasir ve feed'den gercek bir guncelleme gelirse
  (`-r0.1`) apt onu tercih eder, yani depo kilitlenmez.
- Kaldirmak icin: `apt-get remove <paket-adi>`
- Depoyu kaldirmak icin: `rm /etc/apt/sources.list.d/dreambox-feed.list`
