# CloudTrend Sistemi - Kullanım Kılavuzu

Üç TradingView göstergesinden oluşan trend & dönüş takip sistemidir. Crypto, altın/emtia ve hisse/endeks için optimize edilmiştir.

---

## 1. Genel Bakış

Sistem üç göstergeden oluşur. Her biri farklı bir soruya cevap verir:

| Gösterge | Dosya | Sorduğu soru |
|---|---|---|
| **CloudTrend** | `tr_view.pine` | Trend ne yönde? Fiyat hangi seviyede? |
| **OB/OS Tracker** | `ob_os_tracker.pine` | Şu an aşırı alım/satım var mı? Haftalık cycle nerede? |
| **Reversal Scout** | `reversal_scout.pine` | Dip/tepe dönüşü yakın mı? (RSI + Bollinger + Hacim teyidi) |

Üçü ayrı indicator olarak TradingView'e eklenir, aynı chart'ta birlikte çalışır.

> **Not 1:** OB/OS Tracker, opsiyonel haftalık referans çizgisiyle iki katmanlı çalışır: aktif çizgi chart'ın TF'sini gösterir (timing için), ince referans çizgisi haftalık RSI'ı her TF'de gösterir (cycle context için).

> **Not 2:** Reversal Scout, "A'dan Z'ye Borsaya Yatırım Rehberi" teknik analiz bölümündeki dönüş sinyallerini (RSI uçları + Bollinger band dokunuşu + hacim spike) tek bir confluence sinyalinde birleştirir. OB/OS sürekli osilatör gösterirken, Reversal Scout yüksek-güvenli dönüş noktalarını fiyat grafiğinde işaretler.

---

## 2. CloudTrend (`tr_view.pine`)

**Ana panel** — fiyat üstüne overlay edilir. Trend yönü, momentum ve destek/direnç gösterir.

### Görsel Elemanlar

| Element | Anlam |
|---|---|
| Mavi/Cyan bulut | Yükseliş trendi (EMA21 ≥ EMA55) |
| Pembe/Magenta bulut | Düşüş trendi (EMA21 < EMA55) |
| Beyaz çizgi | EMA 200 — uzun dönem trend referansı |
| Yeşil/Kırmızı çizgi | SuperTrend — momentum ve stop-loss seviyesi |
| Turuncu kesikli çizgi | Önceki haftanın yüksek noktası (direnç) |
| Mavi kesikli çizgi | Önceki haftanın düşük noktası (destek) |
| Turuncu/Mavi cross | 50-bar swing high/low |
| Cyan elmas (altta) | Cloud bullish cross — trend yukarı döndü |
| Pembe elmas (üstte) | Cloud bearish cross — trend aşağı döndü |

### Inputlar

| Input | Default | Açıklama |
|---|---|---|
| Asset Type | Crypto | SuperTrend parametrelerini otomatik ayarlar |
| Show S/R Levels | açık | Destek/direnç çizgilerini göster |
| Show Info Table | açık | Sağ üst köşedeki 3 satırlık özet |
| Color Candles by Trend | kapalı | Mumları trend rengine boya |

### Info Table (3 satır)

| Satır | Olası değerler | Anlam |
|---|---|---|
| Status | BULL / BEAR / TRANSITION | Genel durum (bulut + fiyat konumu + SuperTrend uyumu) |
| Price | ABOVE / IN / BELOW CLOUD | Fiyatın buluta göre konumu |
| Strength | STRONG / MODERATE / SIDEWAYS (ADX) | ADX trend gücü |

### Alertler

| Alert | Tetiklenme |
|---|---|
| Cloud Bullish Cross | EMA21 EMA55'i yukarı kesti |
| Cloud Bearish Cross | EMA21 EMA55'i aşağı kesti |
| Price Enters Cloud | Fiyat buluta girdi (kararsız bölge) |
| EMA 200 Break Up | Fiyat 200 EMA üstüne çıktı |
| EMA 200 Break Down | Fiyat 200 EMA altına düştü |
| SuperTrend Bull/Bear Flip | SuperTrend yön değiştirdi |

---

## 3. OB/OS Tracker (`ob_os_tracker.pine`)

**Alt panel** — Kısa vadeli timing + haftalık cycle context, tek panelde. RSI veya Stoch RSI seçilebilir.

### Ne ölçer

İki katmanlı yapı:

- **Aktif çizgi (renkli noktalı):** Chart'ın kendi TF'sindeki osilatör → kısa vade timing için
- **Referans çizgi (ince, opsiyonel):** Her TF'de haftalık osilatörü gösterir → cycle context için
  - Haftalık chart'ta ana çizgiyle örtüşür (görünmez gibi olur)
  - Günlük chart'ta = haftalık RSI yumuşak bir referans çizgisi olarak arkada durur
  - Kendi zone'una göre renklenir: yeşilse haftalık OB, kırmızıysa haftalık OS, beyazsa nötr

### İki katmanın okunması

| Görünüm | Aktif (renkli dots) | Referans (ince çizgi) |
|---|---|---|
| Haftalık chart | Haftalık RSI = Betacycle benzeri | Görünmez (örtüşür) |
| Günlük chart | Günlük RSI = anlık timing | Haftalık RSI = cycle context |
| 4 saatlik | 4h RSI = scalp timing | Haftalık RSI = cycle context |

### Mod karşılaştırması

| Mod | Mantık | Karakter |
|---|---|---|
| **RSI** (default) | Klasik momentum osilatörü | Az ama güçlü sinyal, az gürültü |
| **Stoch RSI** | RSI'ın kendi son N barındaki yüzdelik konumu | Çok daha hareketli, sık sık zone'a girer |

### Skor yorumu (aktif çizgi)

| Değer | Renk | Bölge |
|---|---|---|
| ≥ Overbought | 🟢 Yeşil | Aşırı alım — kısa düzeltme gelebilir |
| Mid | 🔵 Mavi | Nötr bölge |
| ≤ Oversold | 🔴 Kırmızı | Aşırı satım — kısa tepki rallisi gelebilir |

### Inputlar

| Input | Default | Açıklama |
|---|---|---|
| Oscillator Type | RSI | RSI veya Stoch RSI seç |
| RSI Length | 14 | RSI hesaplama periyodu |
| Stoch Length | 14 | Sadece Stoch RSI: RSI'ın yüzdelik konumu için lookback |
| K Smoothing | 3 | Sadece Stoch RSI: K çizgisi yumuşatma |
| Overbought Level | 70 | RSI için 70, Stoch RSI için 80 önerilir |
| Oversold Level | 30 | RSI için 30, Stoch RSI için 20 önerilir |
| Shade OB/OS Zones | açık | Bölgeleri gölgelendir |
| Show Weekly Reference Line | açık | Haftalık osilatörü ince referans çizgisi olarak göster |
| Weekly Reference Opacity | 25 | Referans çizgisinin şeffaflığı (0=opak, 80=neredeyse görünmez) |

### Timeframe önerileri

#### RSI modu

| Timeframe | RSI Len | OB / OS | Notes |
|---|---|---|---|
| Haftalık | 14 | 70 / 30 | Standart — Selcoin Betacycle haftalık'ın yaklaşık karşılığı |
| Günlük (BTC) | 21 | 75 / 25 | Daha az gürültü |
| Günlük (agresif crypto) | 21 | 80 / 20 | Sadece keskin uçlar |
| 4 saatlik | 14 | 75 / 25 | Daha az whipsaw |

#### Stoch RSI modu

| Timeframe | RSI / Stoch / K | OB / OS | Notes |
|---|---|---|---|
| Haftalık | 14 / 14 / 3 | 80 / 20 | Standart |
| Günlük | 14 / 14 / 3 | 80 / 20 | Çok hareketli |
| Günlük (daha az sinyal) | 14 / 21 / 3 | 85 / 15 | Daha selektif |

### Selcoin Betacycle ile ilişki

Selcoin'in **Betacycle** göstergesi, yapılan ters mühendislik sonucu **scaled weekly RSI** çıktı:

```
Betacycle ≈ 1.42 × WeeklyRSI(14) + 72
```

Bu kalibrasyonla:
- Betacycle 110 (kırmızı) ≈ Weekly RSI 30 (klasik oversold)
- Betacycle 170 (yeşil) ≈ Weekly RSI 70 (klasik overbought)
- Betacycle 138 (mavi nötr) ≈ Weekly RSI 47

Yani OB/OS Tracker'ı **haftalık chart**'ta açtığında veya **günlük chart**'tan **haftalık referans çizgisini** izlediğinde, görsel olarak Betacycle haftalık'ın aynısını okursun. Sadece skala 0-100 (70/30 eşik) yerine ~70-210 (Betacycle skalası 110/170).

> **Önemli not:** Tek bir gösterge hem haftalık-yumuşak hem günlük-anlık olamaz — bilgi yoğunluğu farkı. Selcoin'in günlük Betacycle'ı da çok gürültülüdür (kendileri çözememiş). Bu yüzden iki katmanlı yapı: chart TF'i timing için, haftalık referans cycle için.

### RSI mi Stoch RSI mi?

| Tercih | Hangisi |
|---|---|
| Az ama güvenilir sinyal | RSI |
| Sık tepki noktaları yakalamak | Stoch RSI |
| Selcoin / TR crypto sitelerindeki tarz | Stoch RSI |
| Hisse, altın takibi | RSI |
| Crypto scalp/swing | Stoch RSI |

### Alertler

| Alert | Tetiklenme |
|---|---|
| Enter Overbought (Chart) | Chart TF osilatörü overbought eşiğini geçti |
| Exit Overbought (Chart) | Eşiğin altına düştü |
| Enter Oversold (Chart) | Chart TF osilatörü oversold eşiğinin altına düştü |
| Exit Oversold (Chart) | Eşiği geçti |
| Enter Overbought (Weekly) | **Haftalık** osilatör overbought'a girdi — cycle tepesi |
| Exit Overbought (Weekly) | Haftalık overbought'tan çıktı |
| Enter Oversold (Weekly) | Haftalık osilatör oversold'a girdi — cycle dibi |
| Exit Oversold (Weekly) | Haftalık oversold'dan çıktı |

---

## 4. Reversal Scout (`reversal_scout.pine`)

**Ana panel** — fiyat üstüne overlay. Dip/tepe dönüş noktalarını ok + AL/SAT etiketiyle işaretler.

### Ne ölçer

> "Şu an bir tükeniş (exhaustion) noktası mı? Dönüş yakın mı?"

Tek bir gösterge yanıltır; kitap da bunu söylüyor. Bu yüzden üç bağımsız dönüş koşulu birden aranır:

| Koşul | Kitap referansı | Tepe (SAT) | Dip (AL) |
|---|---|---|---|
| **RSI ucu** | s.49: 30/70'de yön değişimi beklenir | RSI ≥ 70 | RSI ≤ 30 |
| **Bollinger dokunuşu** | s.49-50: band sınırından ortalamaya dönüş | High ≥ üst band | Low ≤ alt band |
| **Hacim spike** | s.42: en yüksek hacimler fiyat uçlarında olur | Hacim > ort. × çarpan | Hacim > ort. × çarpan |

Kaç koşulun gerektiği ayarlanabilir (2 veya 3). Opsiyonel MACD teyidi (s.48) ek bir AND kapısı olarak eklenebilir.

### Sinyal mantığı

```
Tepe (SAT) = (≥minConditions tepe koşulu) VE (MACD kapalı VEYA MACD bearish)
Dip (AL)   = (≥minConditions dip koşulu)  VE (MACD kapalı VEYA MACD bullish)
```

- **3 koşul (katı):** Az ama güçlü sinyal, "oturaklı" karakter
- **2 koşul (esnek):** Daha fazla sinyal, bazıları yanlış çıkabilir

Sinyal **3/3** koşulla geldiyse ok **parlak**, **2/3** ile geldiyse **sönük** renkli.

### Inputlar

| Input | Default | Açıklama |
|---|---|---|
| Minimum Conditions to Trigger | 3 | Kaç koşul gerekli (2 veya 3) |
| RSI Length / Overbought / Oversold | 14 / 70 / 30 | RSI ayarları |
| BB Length / StdDev | 20 / 2.0 | Bollinger ayarları |
| Volume MA Length | 20 | Hacim ortalaması periyodu |
| Volume Spike Multiplier | 1.5 | Hacim ortalamanın kaç katı = spike |
| Use MACD Confirmation | kapalı | MACD'yi ek teyit koşulu yap |
| MACD Fast / Slow / Signal | 12 / 26 / 9 | MACD ayarları |
| Show Trend EMA | açık | Trend referans çizgisi (kitap s.46) |
| Trend EMA Length | 200 | Trend EMA periyodu |
| Show Bollinger Bands | açık | BB bantlarını çiz |
| Show AL/SAT Labels | açık | Kapalıyken sadece ok, yazı yok |

### Görsel

| İşaret | Anlam |
|---|---|
| 🔻 Kırmızı ok + "SAT" (bar üstü) | Tepe dönüş sinyali |
| 🔺 Yeşil ok + "AL" (bar altı) | Dip dönüş sinyali |
| Beyaz çizgi | Trend EMA (yön referansı) |
| Gri bantlar | Bollinger bandları |

### Önemli sınırlamalar

- **Hacimsiz/güvenilmez hacimli varlıklar:** Bazı endeks/forex sembollerinde hacim olmayabilir. O zaman hacim koşulu tetiklenmez, max 2 koşul kalır — `Minimum Conditions`'ı 2 yap.
- **Repaint:** Sinyaller bar kapanışından önce intra-bar oluşabilir ve bar kapanana kadar değişebilir. Teyit için bar kapanışını beklemek daha güvenli.
- **Karşı-trend doğası:** Bu bir dönüş avcısıdır, trende karşı sinyal verir. Güçlü trendlerde erken sinyal verebilir — CloudTrend ile birlikte oku (trend hâlâ güçlüyse sinyali zayıf say).

### Timeframe önerileri

| Timeframe | Min Koşul | Volume Mult | Notes |
|---|---|---|---|
| Haftalık | 3 | 1.5 | En güvenilir, az sinyal |
| Günlük (BTC) | 3 | 1.8 | Crypto volatilitesi için biraz sıkı |
| Günlük (daha çok sinyal) | 2 | 1.5 | Esnek mod |
| 4 saatlik | 2 | 1.5 | Daha aktif |

### Alertler

| Alert | Tetiklenme |
|---|---|
| Top Reversal (SAT) | Tepe dönüş sinyali oluştu |
| Bottom Reversal (AL) | Dip dönüş sinyali oluştu |

---

## 5. Üçü Birlikte Nasıl Okunur

### Güçlü AL setup'ı

| Sinyal | Gösterge |
|---|---|
| Cloud Bullish Cross | CloudTrend (trend yukarı döndü) |
| Haftalık RSI oversold'dan çıkıyor (kırmızı referans → beyaza) | OB/OS Tracker (cycle dibinden toparlanma) |
| Chart TF RSI oversold + döndü | OB/OS Tracker (kısa vade satılmış, tepki başlıyor) |
| **AL** sinyali (yeşil ok) | Reversal Scout (RSI+BB+Hacim dip teyidi) |

### Güçlü SAT setup'ı

| Sinyal | Gösterge |
|---|---|
| Cloud Bearish Cross | CloudTrend (trend aşağı döndü) |
| Haftalık RSI overbought (yeşil referans çizgi) | OB/OS Tracker (cycle tepesi) |
| Chart TF RSI overbought + döndü | OB/OS Tracker (kısa vade alınmış) |
| **SAT** sinyali (kırmızı ok) | Reversal Scout (RSI+BB+Hacim tepe teyidi) |

> **En güçlü dönüş:** Reversal Scout 3/3 sinyali + OB/OS haftalık uçta + CloudTrend trend dönüşü aynı bölgede. Üçü hizalanınca tarihsel olarak en güvenilir dip/tepe bölgeleri oluşur.

### Çelişkili durumlar (yaygın)

| Durum | Yorum |
|---|---|
| Cloud BULL + Haftalık OB + Chart OB | Trend yukarı ama her iki TF'de aşırı → yeni long açma, mevcut karı koru |
| Cloud BEAR + Haftalık OS + Chart OS | Trend aşağı ama her iki TF'de aşırı → yeni short açma, kademeli birikim düşünülebilir |
| Cloud TRANSITION + her şey nötr | Sinyal yok, beklemek en doğru aksiyon |
| Cloud BULL + Haftalık nötr + Chart OB | Trend güçlü, kısa vade aşırı → büyük açış için tepki bekle |

---

## 6. Asset & Timeframe Önerileri

### BTC

| TF | CloudTrend Asset | OB/OS Type | OB / OS | RevScout Min Koşul |
|---|---|---|---|---|
| Haftalık | Crypto | RSI 14 | 70 / 30 | 3 |
| Günlük | Crypto | RSI 21 (veya Stoch RSI) | 75 / 25 (veya 80 / 20) | 3 |
| 4 saatlik | Crypto | RSI 14 | 75 / 25 | 2 |

### Altın / Emtia

| TF | CloudTrend Asset | OB/OS Type | OB / OS |
|---|---|---|---|
| Haftalık | Gold/Commodities | RSI 14 | 70 / 30 |
| Günlük | Gold/Commodities | RSI 14 | 70 / 30 |

### Hisse / Endeks

| TF | CloudTrend Asset | OB/OS Type | OB / OS |
|---|---|---|---|
| Haftalık | Stocks/Index | RSI 14 | 70 / 30 |
| Günlük | Stocks/Index | RSI 14 | 65 / 35 (hisseler için daha hassas) |

---

## 7. SSS / Sınırlamalar

**S: Reversal Scout ile OB/OS Tracker'ın ikisi de RSI kullanıyor, neden ayrı?**
H: OB/OS sürekli bir osilatör çizer (her bar bir değer). Reversal Scout ise RSI'ı tek başına değil, Bollinger ve hacimle birlikte ele alıp *sadece üçü hizalandığında* fiyat grafiğine bir dönüş işareti koyar. Biri "durum göstergesi", diğeri "tetik". Birlikte kullanılır.

**S: Reversal Scout hiç sinyal vermiyor / çok az veriyor.**
H: Muhtemelen `Minimum Conditions` 3'te ve hacim koşulu tetiklenmiyor (hacimsiz sembol) ya da piyasa uçlara gelmedi. Çözüm: Min Koşul'u 2 yap, veya Volume Spike Multiplier'ı 1.3'e düşür.

**S: Selcoin'deki Betacycle göstergesiyle aynı mı?**
H: Algoritma birebir değil ama davranış olarak çok yakın. Yapılan analiz sonucu Betacycle ≈ 1.42 × WeeklyRSI + 72 olarak yaklaşık modellendi. OB/OS Tracker'ı haftalık chart'ta açarsan görsel olarak aynısını okuyabilirsin (skala farklı, 0-100 vs ~70-210).

**S: Günlük chart'ta haftalık referans çizgisi neden bazen "donmuş" görünüyor?**
H: Haftalık RSI bir hafta boyunca yavaş değişir, günlük 5-7 bar boyunca aynı veya çok yakın bir değer gösterebilir. Bu normal — bilginin doğası gereği. Cycle context için tasarlandı, anlık değişim için değil.

**S: Günlük sinyaller çok mu yoğun?**
H: Klasik RSI(14) günlük volatil olabilir. Önerimler: RSI Length 21 yap, OB/OS eşiklerini 75/25'e çek. Stoch RSI moduna geçersen daha hareketli ama eşikleri 85/15'e çek. Crypto için daha agresif filtre normal.

**S: Tek bir göstergeyle tepe/dip tahmin edebilir miyim?**
H: Hayır. Hiçbir gösterge tepe/dipleri öngörmez, sadece uyarır. Confluence (CloudTrend + OB/OS chart + haftalık referansın aynı yönde uyarması) en güvenilir yaklaşımdır.

**S: TradingView'deki diğer site ile RSI değerleri tutmuyor.**
H: Farklı veri kaynağı (Binance vs Coinbase), farklı smoothing (RMA vs SMA vs EMA), farklı mum kapanış saati. Standart TradingView RSI Wilder smoothing kullanır.

**S: On-chain göstergelerin yerini tutar mı?**
H: Hayır. MVRV, NUPL gibi metrikler BTC için daha güvenilirdir ama tek asset için çalışır. Bu sistem multi-asset bir alternatiftir.

---

## 8. Sorumluluk Reddi

Bu göstergeler eğitim ve analiz amaçlıdır. Yatırım tavsiyesi değildir. Gerçek para ile kullanmadan önce demo/paper trading ile test edilmesi önerilir. Kripto, altın ve hisse piyasaları yüksek risk barındırır; kaybetmeyi göze alamayacağın parayla işlem yapma.
