# Gün 04 - Günlük Çalışma Raporu

**Tarih:** 23 Temmuz 2026  
**Konu:** Otomatik Fine-Tune Pipeline'ı, Veri Seti Testi ve Proje Yapılandırması  

---

## Günün Özeti

Bugün anotasyon aracının tespit kalitesini artırmaya yönelik iyileştirme fikirlerini detaylı şekilde araştırdım, otomatik fine-tune pipeline scripti yazdım, Mendeley'den açık kaynak glomerulus veri setini indirip pipeline'ı gerçek verilerle test ettim ve fine-tuned modeli v5 aracında denedim. Gün sonunda proje dosya yapısının dağınık hale geldiğini tespit ederek temiz bir organizasyon planı hazırladım.

---

## Yapılan İşler

### 1. İyileştirme Fikirlerinin Araştırılması ve Önceliklendirilmesi
Grounding DINO'nun histoloji görüntülerinde düşük güven skorları (`0.26 – 0.30`) üretmesinin temel nedenini analiz ettim. Modelin bu tür görüntülerde eğitilmemiş olması asıl sorundur. Çözüm olarak dokuz farklı iyileştirme fikrini teknik derinlikte araştırdım:
- **Tiling (Parçalı Tarama):** Büyük görüntüleri örtüşen karolara bölerek küçük yapıları yakalama.
- **Fine-Tune Aktif Öğrenme Döngüsü:** Modelin kendi çıktısını insan doğrulamasından geçirerek sürekli iyileştirme (en kalıcı çözüm).
- **Stain Normalization:** Macenko/Reinhard renk normalizasyonu yöntemleri.
- **Test-Time Augmentation (TTA):** Test anında veri çoğaltma ile tahmin konsensüsü.
- **SAM2 Multimask Seçimi:** 3 maske üretip en iyi IoU'yu seçme.
- **Kutu + Nokta Hibrit Prompt:** DINO kutusu içine negatif/pozitif nokta ekleme.
- **Batch Processing:** Klasör bazlı çoklu görsel işleme.
- **Klavye Onaylı Hızlı İnceleme:** Seri klavye akışı.
- **Profil Kaydet/Yükle:** Parametre setlerini kaydetme.

Her birinin beklenen etkisini, zorluk seviyesini ve uygulama kodunu detaylandırdım.

### 2. Otomatik Fine-Tune Pipeline Scripti (`auto_finetune_pipeline.py`)
Daha önce 8 manuel adımdan oluşan etiketleme ➔ export ➔ merge ➔ fine-tune sürecini tek komutla çalışan bir Python otomasyon scriptine dönüştürdüm:
- Belirtilen klasördeki görseller otomatik olarak Grounding DINO + SAM 2 ile taranıyor.
- Tiling modunda (karolama) parçalı tespitler birleştiriliyor.
- NMS (Non-Maximum Suppression) uygulanıp COCO formatında JSON dosyası üretiliyor.
- Veri seti otomatik olarak `train` / `val` (%80/%20) olarak bölünüyor.
- Fine-tuning işlemi başlatılarak yeni model ağırlıkları `checkpoints/` klasörüne kaydediliyor.

### 3. Veri Seti İndirme ve Pipeline Testi
Mendeley Data'dan **AIDPATH Glomerulus** veri setini indirdim (2340 PNG görüntü — 1170 normal, 1170 sklerotik glomerulus, PAS boyalı, 20x büyütme). Veri setinden 15 görüntü seçip pipeline'ı tiling modunda (`800px` karo, `%20` örtüşme) çalıştırdım. Pipeline başarıyla çalıştı: modeller yüklendi, görüntüler tarandı, COCO JSON üretildi, train/val bölündü ve fine-tune tamamlandı. Fine-tuned modeli v5 aracında *"Custom / fine-tuned"* seçeneğiyle yükleyip test ettim.

### 4. Fine-Tuned Model Test Sonuçları ve Değerlendirme
Fine-tuned model v5'te test edildiğinde glomerülleri doğru konumlarda tespit etti ancak güven skorları hâlâ düşük kaldı (`0.05 – 0.22`). Bunun iki temel nedeni var:
1. Pipeline'ın otomatik ürettiği anotasyonlar insan tarafından düzeltilmeden doğrudan fine-tune'a verildi (eğitim verisi kalitesi yetersiz).
2. Sadece 15 görüntüyle eğitim verisi miktarı yeterli değildi.

Bu deneyim önemli bir bilimsel ders ortaya koydu: **Fine-tune'un işe yaraması için az ama kaliteli, insan tarafından doğrulanmış anotasyon gerekiyor. Otomatik üretilen düşük kaliteli veriyle eğitim, modeli iyileştirmek yerine gürültü öğretir.**

### 5. Proje Dosya Yapısı Planlaması
Çalışmalar sırasında dosyaların farklı klasörlere dağıldığını ve takibin zorlaştığını fark ettim. Bunu çözmek için standart bir proje klasör yapısı tasarladım: ham görüntüler, anotasyonlar, COCO veri setleri, model checkpoint'ları ve sonuçlar ayrı numaralı klasörlerde, her tur (`round`) kendi alt klasöründe tutulacak. Ayrıca bu yapıyı otomatik oluşturan bir kurulum scripti (`setup_project.py`) hazırladım.

---

## Öğrenilen Dersler

- **Kalite > Miktar:** 10 görüntüyü dikkatli ve eksiksiz etiketlemek, 100 görüntüyü otomatik ama hatalı etiketlemekten çok daha değerlidir.
- **Otomasyon ≠ Kısayol:** Pipeline otomasyonu veri üretim hızını artırır ama insan doğrulaması (Human-in-the-Loop) olmadan fine-tune kalitesi düşük kalır.
- **Dosya Organizasyonu Erken Yapılmalı:** Proje başında net klasör yapısı ve isimlendirme kuralları belirlenmezse, birkaç gün içinde süreç takip edilemez hale gelir.

---

## Sonuç ve Sonraki Adımlar

Bugün otomatik pipeline'ı çalışır hale getirdim ve ilk fine-tune denemesini gerçekleştirdim. Sonuçlar beklenen seviyede olmasa da sürecin tamamı uçtan uca test edilmiş oldu. 

Yarın öncelikle temiz proje klasör yapısına geçiş yapılacak, ardından 10-15 görüntü dikkatli şekilde elle anotlanacak ve bu kaliteli veriyle ikinci fine-tune turu (`round 2`) gerçekleştirilecek.

---

## Kullanılan Teknolojiler
`Python 3.12`, `PyTorch 2.11+cu128`, `PyQt5`, `HuggingFace Transformers (Grounding DINO, SAM 2)`, `OpenCV`, `NumPy`, `torchvision (NMS)`, `Mendeley AIDPATH Glomeruli Dataset`

---

## Üretilen Çıktılar
- `auto_finetune_pipeline.py` — Toplu tespit + COCO export + train/val bölme + fine-tune otomasyon scripti.
- `setup_project.py` — Temiz proje klasör yapısını otomatik oluşturan script.
- `proje_yapisi_kilavuz.md` — Klasör yapısı ve iş akışı kılavuzu.

---
[Ana Sayfaya Geri Dön](../README.md) | [Gün 03 Raporu](../Day03/README.md) | [Gün 05 Raporuna Git](../Day05/README.md)

---
*Hazırlayan: Durhasan*
