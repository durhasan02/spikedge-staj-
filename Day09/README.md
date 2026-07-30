# Gün 09 - Günlük Çalışma Raporu

**Tarih:** 30 Temmuz 2026  
**Hazırlayan:** Durhasan Yazğan  
**Konu:** Anotasyon Aracı Kullanılabilirlik İyileştirmeleri, Çizgi (Taper) Aracı ve YOLO Segmentasyon Export Desteği  

---

## Günün Özeti

Bugün elle anotasyonu **daha hızlı ve daha hassas** hale getiren bir dizi araç iyileştirmesi yaptım. Piksel düzeyinde çalışmayı kolaylaştıran canvas iyileştirmeleri, ray gibi ince/lineer yapılar için özel bir **daralan çizgi (taper) aracı** ve elle çizdiğim maskeleri boşa harcamadan **YOLO segmentasyon eğitimine** taşıyan COCO→YOLO-seg export yolunu tamamladım. Ayrıca undo/redo ve sınıf düzeltme gibi kritik kullanılabilirlik hatalarını giderdim. Tüm bu çalışmalar hem bağımsız `annotator.py`'de hem de birleşik **LLM Vision Studio** uygulamasında (gömülü olduğu için otomatik) çalışıyor.

---

## Yapılan İşler

### 1. Canvas Piksel Netliği ve Fırça İyileştirmeleri
Elle etiketlemede piksel düzeyinde çalışmayı kolaylaştırmak için canvas'ı geliştirdim:
- **Keskin piksel görünümü (toggle):** Yakınlaşınca görüntüyü nearest-neighbor ile net kare piksellere çeviren, açılıp kapanabilen bir mod; çok yüksek zoom'da **piksel ızgarası** çiziliyor. Varsayılan yumuşak (aşırı zoom'da blok görünmesin diye), pixel-hassas iş için tek tıkla açılıyor.
- **Kesintisiz fırça/silgi:** Ardışık noktalar arası çizgiyle dolduruldu; hızlı sürüklemede artık kesik/boşluklu iz kalmıyor.
- **Alt + fare tekerleği:** Fırça/silgi boyutunu canvas'tan ayrılmadan değiştirme.

### 2. Maske Temizleme ve İçini Doldurma
- **Clean Selected `[C]`:** Seçili maskedeki **delikleri doldurur** ve **küçük gürültü parçalarını temizler** (morfoloji + bağlı bileşen analizi). Geri alınabilir.
- **İçini Doldur:** Seçili bir maskenin **iç bölgesini** (konveks gövde temelli) **farklı bir sınıfla** yeni maske olarak doldurur — ör. iki metal rayın (rail_raised) arasını tek tıkla `rail_track` ile doldurmak.

### 3. Çizgi (Taper) Aracı — Lineer Yapılar İçin
Ray, tel, çatlak gibi ince ve uzun yapıları fırçayla takip etmek yorucu olduğundan, özel bir **polyline aracı** ekledim:
- Nesnenin merkez çizgisine nokta koyarak çiziyorsun; **kalınlık = fırça boyutu**.
- **Daralan (taper):** İlk uç kalın → son uç ince, yani perspektifle uzaklaşan raya birebir oturuyor.
- `Enter` veya çift-tık bitirir, `ESC` iptal, `Ctrl+Z` son noktayı geri alır.

### 4. YOLO Segmentasyon Export Desteği
Elle çizilen hassas maskelerin segmentasyon eğitimine gidebilmesi için eksik olan yolu tamamladım:
- **Annotator COCO export'u artık `segmentation` (poligon) yazıyor** — önceden yalnızca bounding box saklanıyor, elle çizilen maskeler kayboluyordu. Artık çok-konturlu poligonlar COCO'ya yazılıyor.
- **`coco_to_yolo_format` fonksiyonuna `task=segment`:** COCO segmentasyonundan **normalize YOLO-seg poligon** etiketleri (`class x1 y1 x2 y2 …`) üretiyor.
- **Studio → Fine-Tune → Bölüm 6:** COCO modunda "segment" görevi seçilebiliyor; böylece elle çizilen maskelerle uçtan uca **YOLO11-seg** eğitimi mümkün.

### 5. Kullanılabilirlik Hata Düzeltmeleri
- **Ctrl+Z / Ctrl+Y** artık Studio içinde de çalışıyor. Sorun: gömülü anotasyon penceresi top-level olmadığından menü kısayolu tetiklenmiyordu; çözüm olarak undo/redo'yu doğrudan klavye olayında (keyPressEvent) ele aldım.
- **Çizgi aracında** adım-adım geri alma (eksik olan micro-undo dalı eklendi).
- **Sınıf değiştirme:** Anotasyon listesinde bir maskeye **sağ-tıkla → "Sınıfı değiştir"** ile (çoklu seçim destekli, geri alınabilir) sınıfını değiştirebiliyorum — yanlış sınıfa düşen tespitleri hızlıca düzeltmek için.

---

## Sonuç ve Sonraki Adımlar

Bugünkü çalışmalarla elle anotasyon süreci belirgin biçimde hızlandı ve hassaslaştı; en kritik kazanım, elle çizilen maskelerin artık **YOLO segmentasyon eğitimine uygun** biçimde export edilmesi. Sonraki adımlarda çizgi aracıyla ray verisini toplamayı, **Merge COCO** ile birleştirip bir **YOLO-seg round'u** eğitmeyi ve isteğe bağlı olarak SAM 2 tabanlı maske rötuşu ile kenar-yapışan (intelligent scissors) araçlarını eklemeyi planlıyorum.

---

## Kullanılan Teknolojiler

Python 3.12, PyQt5 (canvas, sinyal/slot, olay yönetimi), OpenCV (kontur, morfoloji, poligon rasterleme), NumPy, Ultralytics (YOLO11-seg), SAM 2 (mevcut segmentasyon), COCO ve YOLO-seg format dönüşümü.

---

## Üretilen Çıktılar

- **`annotator.py` (güncelleme)** — Keskin piksel toggle + piksel ızgarası, kesintisiz fırça/silgi, Alt+tekerlek boyut, maske temizleme (`Clean Selected`), içini doldurma, **Çizgi (taper) aracı**, undo/redo düzeltmeleri (Ctrl+Z/Y), sağ-tık ile sınıf değiştirme, COCO `segmentation` export.
- **`yolo_tools.py` (güncelleme)** — `coco_to_yolo_format`'a segmentasyon (`task=segment`) desteği; COCO poligonlarından YOLO-seg etiketleri.
- **`llm_vision_studio.py` (güncelleme)** — Fine-Tune Bölüm 6'da COCO modunda segment görevi; elle maskelerle YOLO-seg eğitimi.
