# Gün 05 - Günlük Çalışma Raporu

**Tarih:** 24 Temmuz 2026  
**Konu:** Pipeline Düzeltmeleri, TTA Entegrasyonu, Undo/Redo Geliştirmeleri ve Proje Yeniden Yapılandırması  

---

## Günün Özeti

Bugün otomatik pipeline'daki dosya yolu sorunlarını çözdüm, proje klasör yapısını **`llm-vision`** adıyla kuramsal olarak yeniden organize ettim, **Round 1** eğitimini 15 görüntü üzerinde tamamladım ve **TTA (Test-Time Augmentation)** özelliğini test ederek daha iyi tespit sonuçları elde ettim. Ayrıca **Undo/Redo** sistemini görsel butonlar ve araç-içi mikro geri alma desteğiyle genişlettim.

---

## Yapılan İşler

### 1. Pipeline Dosya Yolu Sorunlerinin Çözümü
`auto_pipeline.py` dosyasındaki kalıcı dosya yolu sorununu çözdüm:
- **Sorun:** COCO JSON dosyalarındaki `file_name` alanı sadece dosya adını içeriyordu ama fine-tune scripti görüntüleri JSON'ın yanında arıyordu, görüntüler ise farklı bir alt klasördeydi.
- **Çözüm:** Görüntülerin JSON dosyalarıyla aynı klasöre kopyalanmasını sağladım ve fine-tune dataset sınıfına *çok konumlu arama mantığı* ekledim. Artık script sırasıyla JSON klasörü, `images/` alt klasörü ve üst klasörde görüntü arıyor. Ayrıca gereksiz `images/` alt klasörü oluşturma işlemini kaldırdım.

### 2. Proje Yapısının Yeniden Organizasyonu
Projeyi `C:\Users\durha\Projects\llm-vision` kök klasörü altında temiz bir yapıya taşıdım. Kod dosyalarını anlaşılır isimlerle yeniden adlandırdım:
- `Video04_annotation_tool_v5.py` ➔ **`annotator.py`**
- `Video05_finetune_gdino_v2.py` ➔ **`finetune_tool.py`**
- `auto_finetune_pipeline.py` ➔ **`auto_pipeline.py`**
- `Video02_download_models.py` ➔ **`download_models.py`**

Orijinal depodaki 14 dosyadan yalnızca 5 gerekli dosyayı taşıdım, 9 gereksiz dosyayı (demo notebook'lar, ilgisiz araçlar, eski sürümler) eledim. Otomatik klasör oluşturma scripti ve çift tıkla başlatıcı `.bat` dosyaları hazırladım. Pipeline varsayılan yollarını yeni yapıya uyumlu hale getirdim: `--round` parametresiyle her tur otomatik olarak `data/coco/round_XX` ve `checkpoints/round_XX` klasörlerini kullanıyor.

### 3. Round 1 Eğitimi ve TTA ile İyileştirilmiş Tespit
Mendeley AIDPATH veri setinden 15 glomerulus görüntüsü seçerek **Round 1** pipeline'ını çalıştırdım:
- Tiling modu (`800px` karo) ile **76 anotasyon** üretildi.
- Veri seti `train` / `val` olarak bölündü (`12` / `3`).
- Fine-tune **15 epoch** ile başarıyla tamamlandı.
- **Test-Time Augmentation (TTA):** Ardından TTA özelliğini test ettim. Yatay ve dikey çevirme (horizontal & vertical flip) augmentation'larıyla tespit çalıştırıldığında TTA'sız sonuçlara göre çok daha başarılı tespitler elde edildi. TTA, modelin tek açıdan kaçırdığı nesneleri farklı versiyonlarda yakalayarak `recall` oranını belirgin şekilde artırdı.

### 4. Undo/Redo Sistemi Genişletmeleri
Mevcut `Ctrl+Z` / `Ctrl+Y` undo/redo sistemi üzerine iki önemli geliştirme tasarladım ve spesifikasyonlarını hazırladım (`undo_buttons_micro_spec.md`):
1. **Görsel Araç Çubuğu Butonları:** Araç çubuğuna görsel geri al/ileri al butonları eklendi — kırmızı **"◀ Geri Al"** ve yeşil **"İleri Al ▶"** butonları, işlem sayacı ve tooltip (ipucu) ile son işlemin açıklamasını gösteriyor.
2. **Araç-İçi Mikro Undo/Redo Sistemi:** Vertex editing sırasında köşe sürükleme, yeni köşe ekleme ve köşe silme işlemleri adım adım geri alınabiliyor. Aynı sistem poligon çizimi (son köşeyi geri al), fırça boyama (son darbeyi geri al) ve multi-point (son noktayı geri al) araçları için de geçerli. 

Mikro undo, aktif araç tamamlandığında veya iptal edildiğinde temizleniyor, sonuç ana undo yığınına (`main undo stack`) tek bir işlem olarak kaydediliyor.

---

## Günün Öne Çıkan Başarıları

- **TTA Etkisi:** Test anında simetri çevrimleri kullanarak, ek eğitim gerektirmeden modelin tespit etme hassasiyeti (`recall`) artırıldı.
- **Granüler Undo/Redo:** Fırça darbesi veya poligon köşesi düzeyinde mikro geri alma akışı sayesinde, anotasyon sırasında yapılan küçük hatalarda tüm maskeyi baştan çizme derdi ortadan kaldırıldı.

---

## Sonuç ve Sonraki Adımlar

Gün sonunda proje temiz bir klasör yapısına kavuştu, pipeline dosya yolu sorunları çözüldü ve TTA ile tespit kalitesi artırıldı. Undo/redo sistemi hem görsel butonlar hem de araç-içi mikro geri alma ile güçlendirildi. 

Sonraki adımlarda undo/redo iyileştirmelerini koda tam entegre etmeyi, 10-15 görüntüyü dikkatli şekilde elle anotlayarak kaliteli eğitim verisi oluşturmayı ve bu veriyle **Round 2** fine-tune işlemini gerçekleştirerek skor artışını ölçmeyi planlıyorum.

---

## Kullanılan Teknolojiler
`Python 3.12`, `PyTorch 2.11+cu128`, `PyQt5`, `HuggingFace Transformers (Grounding DINO, SAM 2)`, `OpenCV`, `NumPy`, `torchvision (NMS)`, `Mendeley AIDPATH Glomeruli Dataset`

---

## Üretilen Çıktılar
- `auto_pipeline.py` *(güncelleme)* — Dosya yolu düzeltmeleri, round sistemi, yeni proje yapısına uyum.
- `setup_clean_project.py` — Temiz proje yapısını oluşturan, dosyaları yeniden adlandıran ve başlatıcılar üreten script.
- `undo_buttons_micro_spec.md` — Görsel undo/redo butonları ve araç-içi mikro geri alma spesifikasyonu.
- `data/coco/round_01/` — 15 görüntü, 76 anotasyon içeren `train.json` ve `val.json` (ilk tur eğitim verisi).

---
*Hazırlayan: Durhasan*
