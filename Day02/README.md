# Gün 02 - Günlük Çalışma Raporu

**Tarih:** 21 Temmuz 2026  
**Konu:** Anotasyon Aracı Analizi, Özellik Tasarımı ve v5 Geliştirmesi  

---

## Günün Özeti

Bugün tarım odaklı nesne tespiti projesi kapsamında üç ana çalışma yürüttüm: DigitalSreeni'nin `llm-vision-for-scientists` projesinin kaynak kodunu ve video serisini inceleyerek çalışma mantığını anladım, mevcut araçtaki eksiklikleri ve iyileştirme alanlarını belirledim, ardından üç yeni özelliğin teknik spesifikasyonunu hazırlayarak **v5 sürümünü** geliştirdim.

---

## Yapılan İşler

### 1. Proje Deposu Kurulumu ve Video Analizi
`bnsreenu/llm-vision-for-scientists` GitHub deposunu klonladım. Bu depo, DigitalSreeni'nin video serisindeki tüm kodları içeriyor: Grounding DINO ile bounding box tespiti, DINO + SAM 2 segmentasyon pipeline'ı, PyQt5 tabanlı interaktif anotasyon aracı (`annotation_tool_v4.py`), fine-tuning scripti ve RAG tabanlı literatür aracı. Video serisini izleyerek aracın çalışma mantığını kavradım. `annotation_tool_v4.py` dosyasını (~2400 satır) çalıştırıp test ettim. Modelleri indirip glomerulus sınıfı üzerinde detection ve segmentasyon akışını doğruladım.

### 2. Mevcut Aracın Değerlendirilmesi ve İyileştirme Planı
Mevcut v4 aracını analiz ederek eksik ve iyileştirilebilir alanları belirledim. Tespit ettiğim başlıca sorunlar:
- **Tek görüntü sınırlaması:** Batch (toplu klasör) processing desteği yok.
- **Manuel düzeltme araçlarının kısıtlılığı:** Sadece nokta tıklama ile maske ekleme/silme yapılabiliyor.
- **Güven skoru filtresi eksikliği:** Otomatik tespit sonuçlarında güven bazlı filtreleme bulunmuyor.
- **Kenar düzenleme kısıtı:** Maske kenarlarını poligon düzeyinde düzenleme imkanı yok.
- **İlerleme takibi:** İlerleme göstergesi (progress bar/indicator) eksikliği.

Ayrıca tarıma özel prompt şablonları, **SAHI entegrasyonu** (küçük nesne tespiti için) ve geri alma (`Undo`) özelliği gibi ek iyileştirmeler de planladım. Tüm önerileri zorluk, katkı ve akademik değer açısından önceliklendirdim.

### 3. Üç Öncelikli Özelliğin Teknik Spesifikasyonu
En yüksek öncelikli üç özellik için detaylı teknik spesifikasyon dokümanı hazırladım:
1. **Güven Puanı Bazlı İnsan İnceleme Kuyruğu (Human-in-the-Loop Review):** Detection sonrası maskeler güven puanına göre otomatik kabul, inceleme veya düşük güven olarak ayrılıyor.
2. **Maske Kenar Düzenleme (Vertex Editing):** Mevcut maskelerin poligon köşe noktalarını sürükleyerek hassas düzeltme imkanı.
3. **Çoklu Seçim Araçları:** SAM-box, SAM-multi-point (pozitif/negatif nokta), manuel poligon, fırça, silgi ve vertex düzenleme olmak üzere 9 farklı araç. 

Spesifikasyonda her özelliğin GUI tasarımı, veri yapıları, PyQt5 widget kodları, sinyal bağlantıları ve test senaryoları yer alıyor.

### 4. `annotation_tool_v5.py` Geliştirmesi ve Kod İncelemesi
Hazırladığım spesifikasyonu kullanarak **v5 sürümünü** ürettim. Ortaya çıkan dosya **3488 satır** Python kodundan oluşuyor ve v4'e göre yaklaşık **1100 satır eklenti** içeriyor. v5 kodunu bölüm bölüm inceleyerek 12 ana blokta detaylı bir kod kılavuzu oluşturdum. Bu kılavuzda her fonksiyonun ne yaptığı, neden var olduğu ve nerede çağrıldığı açıklanıyor. 

Özellikle şu kritik bloklar üzerine yoğunlaştım:
- `DetectionWorker.run()`: Tespit pipeline'ının kalbi.
- `_on_det_done()`: Güven bazlı ayırma mantığı.
- `ImageCanvas.paintEvent()`: Tüm görsel çizim işleyicisi.
- Araç yönlendirme sistemi (`_on_canvas_pressed` ➔ aktif araca dallanma).

---

## v5'te Eklenen Özellikler Özeti

- **HITL Review Queue:** Detection sonrası düşük güvenli tespitler inceleme kuyruğuna giriyor. Kullanıcı tek tek veya toplu olarak kabul/ret/düzenleme yapabiliyor. Canvas'ta sarı (orta güven) ve kırmızı (düşük güven) kenarlıklarla görsel geri bildirim sağlanıyor.
- **Vertex Editing:** Maskelerin poligon köşe noktaları sürüklenebilir handle'larla düzenlenebiliyor. Çift tıklama ile yeni köşe ekleme, sağ tıklama ile köşe silme destekleniyor. `Enter` ile onay, `ESC` ile iptal edilebiliyor.
- **Çoklu Araç Sistemi:** 9 farklı anotasyon aracı eklendi:
  - View (`V`), SAM-point (`P`), SAM-box (`B`), Multi-point (`M`), Polygon (`G`), Brush (`R`), Eraser (`E`), Edit vertices (`T`), Delete (`D`).
  - Klavye kısayolları ve fırça boyutu (`brush size slider`) entegre edildi.

---

## Sonuç ve Sonraki Adımlar

Gün sonunda mevcut aracın çalışma mantığını kavramış, eksikliklerini belirlemiş ve üç kritik özelliği hem tasarlamış hem de implemente etmiş olduk. 

Sonraki adımlarda v5 aracını tarım görüntüleri üzerinde test etmeyi, batch processing ve SAHI entegrasyonu gibi kalan iyileştirmeleri eklemeyi ve fine-tuning ile aktif öğrenme döngüsünü kurmayı planlıyorum.

---

## Kullanılan Teknolojiler
`Python 3.12`, `PyTorch 2.11+cu128`, `PyQt5`, `HuggingFace Transformers (Grounding DINO, SAM 2)`, `OpenCV`, `NumPy`, `torchvision (NMS)`, `llm-vision-for-scientists (bnsreenu)`

---

## Üretilen Çıktılar
- `annotation_tool_v5_spec.md` — 3 özelliğin teknik spesifikasyon dokümanı.
- `annotation_tool_v5.py` — 3488 satırlık geliştirilmiş masaüstü anotasyon aracı.
- `annotation_tool_v5_kod_kilavuzu.md` — Kodun bölüm bölüm açıklamalı teknik kılavuzu.

---
*Hazırlayan: Durhasan*
