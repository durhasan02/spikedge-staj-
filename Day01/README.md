# Gün 01 - Günlük Çalışma Raporu

**Tarih:** 20 Temmuz 2026  
**Konu:** Grounding DINO + SAM 2 Tabanlı Anotasyon Aracı Kurulumu  

---

## Günün Özeti

Bugün tarım görüntülerinde nesne tespiti ve segmentasyonu için **Grounding DINO** ve **SAM 2** modellerini içeren bir masaüstü anotasyon aracını kurdum ve çalıştırdım. Amaç, bitki hastalık tespiti projesi için metin tabanlı otomatik anotasyon pipeline'ı oluşturmaktı.

---

## Yapılan İşler

### 1. Python Sanal Ortam ve GPU Yapılandırması
Çalışma ortamı olarak `Python 3.12` üzerinde bir sanal ortam (`venv`) oluşturdum. Makinemdeki **NVIDIA RTX 5060 Laptop GPU**'yu kullanabilmek için PyTorch'un `CUDA 12.8` destekli sürümünü kurdum. İlk denemede CUDA 12.4 ile kurulum yaptığım için GPU kernel uyumsuzluğu yaşadım — RTX 5060 Blackwell mimarisinde olduğundan eski CUDA sürümleriyle çalışmıyordu. CUDA Toolkit 12.8'i ayrıca kurup `PATH` ayarlarını yaptıktan sonra GPU doğrulaması başarıyla geçti.

### 2. DigitalSreeni Image Annotator Kurulumu (SAM 2)
İlk olarak pip üzerinden `digitalsreeni-image-annotator` aracını kurdum. Bu araç **SAM 2 (Segment Anything Model 2)** ile yarı-otomatik segmentasyon yapabiliyor. Kurulum sırasında `imagecodecs` kütüphanesiyle uyumluluk sorunu ve `PyQt5` ile `torch` arasında DLL çakışması gibi sorunlarla karşılaştım; bunları sürüm sabitlemesi ve özel başlatıcı script ile çözdüm. SAM 2 `tiny`, `small`, `base` ve `large` modellerini test ettim. Glomerulus sınıfı üzerinde manuel kutu çizerek maske üretimi başarılı şekilde çalıştı.

### 3. Grounding DINO + SAM 2 Pipeline Testi
Grounding DINO modelini **HuggingFace Transformers** kütüphanesi üzerinden kurdum (`IDEA-Research/grounding-dino-tiny`). C++ derleyici gerektiren kaynak koddan derleme yerine bu yol çok daha pratik oldu. Bir test scripti yazarak Grounding DINO ile metin prompt tabanlı nesne tespiti ve ardından SAM 2 ile segmentasyon yapan bir pipeline oluşturdum. `"person . bus ."` prompt'uyla bir test görüntüsünde başarılı tespit ve maske üretimi gerçekleştirdim.

### 4. LLM Vision for Scientists — Tam Proje Kurulumu
Son olarak `bnsreenu/llm-vision-for-scientists` GitHub deposunu klonladım. Bu depo, DigitalSreeni'nin video serisindeki tüm kodları içeriyor: Grounding DINO ile bounding box tespiti (Video 2), DINO + SAM 2 segmentasyon pipeline'ı (Video 3), PyQt5 tabanlı interaktif anotasyon aracı (Video 4), fine-tuning aracı (Video 5), RAG tabanlı literatür destekli tespit (Video 6) ve SAM3 karşılaştırması (Video 7). `annotation_tool_v4.py` dosyasını çalıştırarak Grounding DINO + SAM 2 entegre GUI'yi başarıyla açtım.

---

## Karşılaşılan Sorunlar ve Çözümler

- **`imagecodecs` uyumsuzluğu:** `czifile` kütüphanesi yeni imagecodecs sürümüyle çalışmıyordu. `imagecodecs<2025` sürümünü sabitleyerek çözdüm.
- **CUDA kernel hatası:** RTX 5060 (Blackwell) için CUDA 12.8 ve uyumlu PyTorch gerekiyordu. `torch+cu128` kurarak çözdüm.
- **DLL yükleme hatası (`c10.dll`):** CUDA Toolkit 12.8 kurulumu ve `PATH` yapılandırmasıyla çözüldü.
- **PyQt-torch DLL çakışması:** `.exe` üzerinden çalıştırınca hata veriyordu. `torch`'u PyQt'den önce import eden bir `launcher.py` ile çözdüm.
- **C++ derleyici eksikliği:** Grounding DINO kaynak koddan derlenemedi. HuggingFace Transformers API üzerinden derleme gerektirmeden kullandım.

---

## Sonuç ve Sonraki Adımlar

Gün sonunda hem SAM 2 tabanlı yarı-otomatik anotasyon aracı hem de Grounding DINO + SAM 2 tam otomatik pipeline'ı çalışır duruma geldi. Ayrıca bnsreenu'nun tam proje deposunu da çalıştırarak metin tabanlı otomatik tespit + segmentasyon + manuel düzeltme + COCO export akışını test ettim. 

Sonraki adımlarda bu pipeline'ı tarım görüntüleri üzerinde test etmeyi, sınıf bazlı threshold optimizasyonu yapmayı ve gerekirse Grounding DINO'yu fine-tune etmeyi planlıyorum.

---

## Kullanılan Teknolojiler
`Python 3.12`, `PyTorch 2.11+cu128`, `CUDA Toolkit 12.8`, `Grounding DINO (IDEA-Research, HuggingFace)`, `SAM 2 (Meta AI, Ultralytics)`, `PyQt5`, `digitalsreeni-image-annotator`, `llm-vision-for-scientists`


---
*Hazırlayan: Durhasan*
