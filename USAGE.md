# CloudTrend Sistemi - Kullanım Kılavuzu

Üç ayrı TradingView göstergesinden oluşan bir trend & cycle takip sistemidir. Crypto, altın/emtia ve hisse/endeks için optimize edilmiştir.

---

## 1. Genel Bakış

Sistem üç katmanlı zaman ölçeği üzerinde çalışır. Her gösterge farklı bir soruya cevap verir:

| Gösterge | Dosya | Zaman ölçeği | Sorduğu soru |
|---|---|---|---|
| **CloudTrend** | `tr_view.pine` | Anlık | Trend ne yönde? Fiyat hangi seviyede? |
| **CycleScope** | `cycle_scope.pine` | Uzun vade (aylar) | Büyük döngünün neresindeyiz? |
| **OB/OS Tracker** | `ob_os_tracker.pine` | Kısa vade (günler) | Şu an aşırı alım/satım var mı? |

Üçü ayrı indicator olarak TradingView'e eklenir, aynı chart'ta birlikte çalışır.

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

## 3. CycleScope (`cycle_scope.pine`)

**Alt panel** — uzun dönem döngü pozisyonunu 0-100 skor olarak gösterir.

### Ne ölçer

> "Şu an, geçmiş döngüye göre neredeyiz?"

Üç bileşeni opsiyonel olarak ağırlıklandırılmış şekilde birleştirir (composite mode), veya sadece Mayer ile çalışır (sade mode).

- **Mayer** (default %50): Fiyatın uzun dönem ortalamasından sapması (klasik Mayer Multiple mantığı)
- **MTF RSI** (default %25): Günlük + Haftalık + Chart RSI ortalaması — çok katmanlı momentum
- **Momentum/ROC** (default %25): Normalize edilmiş rate-of-change

Sonuç 0-100 arası bir skora çevrilir. **Overbought/oversold değildir** — döngü pozisyonudur.

### Skor yorumu

| Skor | Renk | Bölge | Anlam |
|---|---|---|---|
| 80-100 | 🟢 Yeşil | TOP | Geçmiş döngüde böyle yerlerden düşüş başlamıştı — dikkat |
| 60-80 | 🔵 Mavi | Üst orta | Trend yukarı ama tepe yaklaşıyor |
| 40-60 | 🔵 Mavi | Nötr | Döngünün belirsiz/geçiş bölgesi |
| 20-40 | 🔵 Mavi | Alt orta | Düşüş hâkim ama dip yaklaşıyor |
| 0-20 | 🔴 Kırmızı | BOTTOM | Geçmiş döngüde böyle yerlerden yükseliş başlamıştı — fırsat |

### Trend filter — Güçlü / Zayıf ayrımı

Trend filter açıkken (default), zone'a girilse bile trend uyumu kontrol edilir:

| Durum | Anlam | Renk |
|---|---|---|
| Top + downtrend | **STRONG TOP** — trend dönüş başladı, cycle teyit ediyor | 🟢 Parlak yeşil |
| Top + uptrend | **WEAK TOP** — cycle aşırı ama trend hâlâ yukarı, erken olabilir | 🟢 Sönük yeşil |
| Bottom + uptrend | **STRONG BOTTOM** — trend dönüş başladı, cycle teyit ediyor | 🔴 Parlak kırmızı |
| Bottom + downtrend | **WEAK BOTTOM** — cycle aşırı ama trend hâlâ aşağı, erken olabilir | 🔴 Sönük kırmızı |

Trend filter kapalıyken bu ayrım yok, ham zone gösterilir.

### Cooldown

Aynı yöndeki sinyaller arasında minimum bar mesafesi (default 60). Cycle göstergesinin doğası gereği zone'da uzun süre kalıp her bar yeniden tetikleme spam'ini önler.

### Inputlar

#### Cycle grubu
| Input | Default | Açıklama |
|---|---|---|
| Auto-Adapt to Timeframe | açık | Chart TF'ine göre longMA ve lookback otomatik seçilir |
| Long MA Length (manual) | 200 | Manuel mod: uzun MA periyodu |
| Lookback (manual) | 300 | Manuel mod: normalizasyon penceresi |
| Normalization | Z-Score | Z-Score / Percentile Rank seçimi |
| Output Smoothing | 8 | HMA yumuşatma periyodu (1 = yumuşatma yok) |

#### Composite grubu
| Input | Default | Açıklama |
|---|---|---|
| Use Composite Scoring | açık | Açık: 3 bileşen birleşik. Kapalı: sadece Mayer (sade mod) |
| Mayer weight | 0.50 | Mayer bileşeni ağırlığı |
| MTF RSI weight | 0.25 | RSI bileşeni ağırlığı |
| Momentum/ROC weight | 0.25 | Momentum bileşeni ağırlığı |

#### MTF RSI grubu
| Input | Default | Açıklama |
|---|---|---|
| RSI Length | 14 | RSI periyodu |
| Use Daily RSI | açık | Günlük TF RSI'ı dahil et |
| Use Weekly RSI | açık | Haftalık TF RSI'ı dahil et |
| Use Current TF RSI | açık | Chart'ın kendi TF RSI'ı dahil et |

#### Zones / Trend Filter / Cooldown grupları
| Input | Default | Açıklama |
|---|---|---|
| Top Zone | 80 | Tepe bölgesi eşiği |
| Bottom Zone | 20 | Dip bölgesi eşiği |
| Apply Trend Filter | açık | Strong/Weak ayrımı yap |
| Trend EMA Length | 100 | Trend referans EMA |
| Enable Signal Cooldown | açık | Sinyal spam'ini önle |
| Min Bars Between Same-Direction Signals | 60 | Cooldown süresi |

### Auto-Adapt parametre tablosu

| Chart TF | Long MA | Lookback |
|---|---|---|
| Aylık | 12 | 60 |
| Haftalık | 50 | 200 |
| Günlük | 200 | 365 |
| Intraday | 100 | 500 |

### Normalization yöntemleri

| Mod | Mantık | Avantaj | Dezavantaj |
|---|---|---|---|
| **Z-Score** (default) | "Ortalamadan kaç sigma sapma" | Kısa veride çalışır, asla boş kalmaz | Ekstrem değerler clamp edilir |
| **Percentile Rank** | "Geçmişin yüzde kaçından yüksek" | Daha intuitif okuma | Az veride na üretir |

> Percentile modunda yeterli veri yoksa otomatik z-score'a düşer.

### Divergence

| Tip | Sinyal | Anlam |
|---|---|---|
| 🔻 Bearish (turuncu üçgen aşağı) | Fiyat HH, cycle LH | Momentum tükendi, tepe yakın olabilir |
| 🔺 Bullish (cyan üçgen yukarı) | Fiyat LL, cycle HL | Düşüş gücünü kaybetti, dip yakın olabilir |

**Önemli:** Pivot tespiti `divLookback` (default 5) bar gecikme gerektirir. Üçgen ekrana geldiğinde fiili pivot anından 5 bar geç olur — bu kaçınılmaz, pivotun doğası gereği.

### Reversal candles (zone'da dönüş mumları)

Divergence'tan farklı, daha hızlı bir sinyal. Cycle skoru zone yakınında (default ±8 puan tolerans) iken klasik dönüş mum formasyonu görülürse elmas işaretiyle gösterilir. Divergence pivot beklerken, reversal candle anında ateşlenir.

| İşaret | Patern | Anlam |
|---|---|---|
| 🔶 Turuncu elmas (top zone) | Shooting star (uzun üst fitil) veya bearish engulfing | Alıcı tükeniyor, tepe yakın |
| 🔷 Cyan elmas (bottom zone) | Hammer (uzun alt fitil) veya bullish engulfing | Satıcı tükeniyor, dip yakın |

**Pattern tanımları:**
- **Shooting star / Hammer**: Üst/alt fitil gövdeden en az `wickRatio` (default 2.0) kat uzun, gövde son 20 barın ortalamasından küçük
- **Bearish engulfing**: Bugünkü kırmızı mum, dünkü yeşil mumu tamamen sarıyor
- **Bullish engulfing**: Bugünkü yeşil mum, dünkü kırmızı mumu tamamen sarıyor

| Input | Default | Açıklama |
|---|---|---|
| Show Reversal Candles in Zone | açık | Elmas işaretlerini göster |
| Zone Tolerance (points) | 8 | Zone'a girmemiş ama yakındaki mumları yakalamak için tampon |
| Min Wick / Body Ratio | 2.0 | Fitilin gövdeye oranı (shooting star / hammer için) |

### Alertler

| Alert | Tetiklenme |
|---|---|
| Enter Top Zone | Effective top zone'a girildi (trend filter + cooldown sonrası) |
| Enter Bottom Zone | Effective bottom zone'a girildi |
| Bearish Divergence | Olası tepe formasyonu (pivot bazlı, gecikmeli) |
| Bullish Divergence | Olası dip formasyonu (pivot bazlı, gecikmeli) |
| Top Reversal Candle | Top zone'da bearish dönüş mumu (anlık) |
| Bottom Reversal Candle | Bottom zone'da bullish dönüş mumu (anlık) |

---

## 4. OB/OS Tracker (`ob_os_tracker.pine`)

**Alt panel** — RSI tabanlı kısa vadeli aşırı alım/satım göstergesi.

### Ne ölçer

> "Son 14 barda fiyat çok mu hızlı arttı/azaldı?"

Klasik RSI'ı renkli noktalarla görselleştirir. CycleScope ile karıştırılmamalı:

| | OB/OS Tracker | CycleScope |
|---|---|---|
| Ölçer | Kısa vade momentum (14 bar) | Uzun vade döngü pozisyonu |
| Zaman | Günler-haftalar | Aylar-yıllar |
| Kullanım | Giriş/çıkış zamanlaması | Genel pozisyonlanma, risk yönetimi |

### Skor yorumu

| RSI | Renk | Bölge |
|---|---|---|
| ≥ 70 | 🟢 Yeşil | Overbought — kısa düzeltme gelebilir |
| 30-70 | 🔵 Mavi | Mid — nötr bölge |
| ≤ 30 | 🔴 Kırmızı | Oversold — kısa tepki rallisi gelebilir |

### Inputlar

| Input | Default | Açıklama |
|---|---|---|
| RSI Length | 14 | RSI periyodu |
| Overbought Level | 70 | Overbought eşiği |
| Oversold Level | 30 | Oversold eşiği |
| Shade OB/OS Zones | açık | Bölgeleri gölgelendir |

### Timeframe önerileri

| Timeframe | RSI Len | OB / OS | Notes |
|---|---|---|---|
| Haftalık (klasik) | 14 | 70 / 30 | Standart, az ama güçlü sinyal |
| Günlük (BTC) | 21 | 75 / 25 | Daha az gürültü |
| Günlük (agresif crypto) | 21 | 80 / 20 | Sadece keskin uçlar |
| 4 saatlik | 14 | 75 / 25 | Daha az whipsaw |

### Alertler

| Alert | Tetiklenme |
|---|---|
| Enter Overbought | RSI 70'i geçti |
| Exit Overbought | RSI 70'in altına düştü |
| Enter Oversold | RSI 30'un altına düştü |
| Exit Oversold | RSI 30'u geçti |

---

## 5. Üçü Birlikte Nasıl Okunur

Tek bir gösterge yalan söyleyebilir; üçü birden aynı şeyi söylediğinde sinyal güçlüdür. Confluence örnekleri:

### Güçlü AL setup'ı

| Sinyal | Gösterge |
|---|---|
| Cloud Bullish Cross | CloudTrend (trend yukarı döndü) |
| Bottom Zone (skor < 20) | CycleScope (cycle dibinde) |
| RSI Oversold + döndü | OB/OS Tracker (kısa vade satılmış, tepki başlıyor) |
| **+** Bullish Divergence | CycleScope (momentum dönüyor) |

### Güçlü SAT setup'ı

| Sinyal | Gösterge |
|---|---|
| Cloud Bearish Cross | CloudTrend (trend aşağı döndü) |
| Top Zone (skor > 80) | CycleScope (cycle tepesinde) |
| RSI Overbought + döndü | OB/OS Tracker (kısa vade alınmış, dönüş başlıyor) |
| **+** Bearish Divergence | CycleScope (momentum tükeniyor) |

### Çelişkili durumlar (yaygın)

| Durum | Yorum |
|---|---|
| Cloud BULL + Cycle TOP + RSI OB | Trend yukarı ama tepe bölgesi → yeni long açma, mevcut karı koru |
| Cloud BEAR + Cycle BOTTOM + RSI OS | Trend aşağı ama dip bölgesi → yeni short açma, kademeli birikim düşünülebilir |
| Cloud TRANSITION + Cycle MID + RSI nötr | Sinyal yok, beklemek en doğru aksiyon |
| Cloud BULL + Cycle MID + RSI OB | Trend güçlü, kısa vade aşırı → büyük açış için tepki bekle |

---

## 6. Asset & Timeframe Önerileri

### BTC

| TF | CloudTrend Asset | CycleScope | OB/OS |
|---|---|---|---|
| Haftalık | Crypto | Auto-adapt (longMA 50, lookback 200) | 14 / 70-30 |
| Günlük | Crypto | Auto-adapt (longMA 200, lookback 365) | 21 / 75-25 |
| 4 saatlik | Crypto | Manuel: longMA 100, lookback 500 | 14 / 75-25 |

### Altın / Emtia

| TF | CloudTrend Asset | CycleScope | OB/OS |
|---|---|---|---|
| Haftalık | Gold/Commodities | Auto-adapt | 14 / 70-30 |
| Günlük | Gold/Commodities | Auto-adapt | 14 / 70-30 |

### Hisse / Endeks

| TF | CloudTrend Asset | CycleScope | OB/OS |
|---|---|---|---|
| Haftalık | Stocks/Index | Auto-adapt | 14 / 70-30 |
| Günlük | Stocks/Index | Auto-adapt | 14 / 65-35 (hisseler için daha hassas) |

---

## 7. SSS / Sınırlamalar

**S: CycleScope ve OB/OS aynı şey mi?**
H: Hayır. OB/OS kısa vade momentum (14 bar), CycleScope uzun vade pozisyon (yüzlerce bar). Farklı sorulara cevap verirler.

**S: Selcoin/diğer sitedeki "Betacycle" göstergesiyle aynı mı?**
H: Hayır. Betacycle proprietary, algoritması açık değil. CycleScope benzer davranışı üreten açık kaynak alternatif — birebir aynı sayılar olmaz.

**S: Haftalık BTC'de CycleScope çizmiyor.**
H: Auto-Adapt'in açık olduğundan emin ol. Kapalıysa lookback değerini düşür (200 veya altı), veri yeterli değil. Z-Score moduna geç, percentile yerine.

**S: Divergence üçgeni neden geç geliyor?**
H: Pivot tespiti `divLookback` (default 5) bar gecikme gerektirir, kaçınılmaz. Daha hızlı için lookback'i düşür ama gürültü artar.

**S: TradingView'deki diğer site ile RSI değerleri tutmuyor.**
H: Farklı veri kaynağı (Binance vs Coinbase), farklı smoothing (RMA vs SMA vs EMA), farklı mum kapanış saati. Standart TradingView RSI Wilder smoothing kullanır.

**S: On-chain göstergelerin yerini tutar mı?**
H: Hayır. MVRV, NUPL gibi metrikler BTC için daha güvenilirdir ama tek asset için çalışır. Bu sistem multi-asset bir alternatiftir.

**S: Tek bir göstergeyle tepe/dip tahmin edebilir miyim?**
H: Hayır. Hiçbir gösterge tepe/dipleri öngörmez, sadece uyarır. Confluence (3 göstergenin birlikte uyarması) en güvenilir yaklaşımdır. Dürüst söyleyen herkes bunu kabul eder.

---

## 8. Sorumluluk Reddi

Bu göstergeler eğitim ve analiz amaçlıdır. Yatırım tavsiyesi değildir. Gerçek para ile kullanmadan önce demo/paper trading ile test edilmesi önerilir. Kripto, altın ve hisse piyasaları yüksek risk barındırır; kaybetmeyi göze alamayacağın parayla işlem yapma.
