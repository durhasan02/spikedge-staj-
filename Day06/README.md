# Gün 06 - Günlük Çalışma Raporu

**Tarih:** 27 Temmuz 2026  
**Konu:** Canvas İyileştirmeleri ve Birleşik Araç Mimarisi Tasarımı (LLM Vision Studio)  

---

## Günün Özeti

Bugün iki büyük mimari tasarım çalışması yaptım: birincisi, anotasyon aracındaki canvas bileşenini farklı görüntü boyutlarına uyumlu, döndürülebilir ve profesyonel kontrollere sahip hale getirmek için kapsamlı bir iyileştirme spesifikasyonu hazırladım. İkincisi ve en kritik olanı, şu ana kadar üç ayrı script olarak çalışan araçları (anotasyon, pipeline, fine-tune) tek bir tab-bazlı masaüstü uygulamasında birleştiren **LLM Vision Studio** mimarisini tasarladım.

---

## Yapılan İşler

### 1. Canvas İyileştirme Spesifikasyonu (6 Özellik)
Anotasyon aracının canvas bileşeninde farklı boyutlardaki görüntülerin düzgün yerleşmemesi ve görüntü kontrollerinin yetersizliği sorunlarını çözmek için altı özellik içeren detaylı bir spesifikasyon hazırladım:
- **Akıllı Sığdırma Modları:** *Fit All* (tüm görüntü sığar), *Fit Width* (genişliğe sığdır) ve `1:1` gerçek boyut modları; pencere yeniden boyutlandırıldığında otomatik uyum.
- **Görüntü Döndürme:** 90° adımlarla saat yönünde ve tersine döndürme; anotasyonlar orijinal koordinatlarda kalırken sadece görüntülemenin dönmesi.
- **Gelişmiş Zoom / Pan:** İmleç merkezli zoom (Google Maps tarzı — imleç altındaki piksel yerinde kalır), orta tuş ile sürükle-kaydır.
- **HUD Bilgi Paneli:** Canvas köşelerinde yarı-şeffaf bilgi kutuları — zoom yüzdesi, görüntü boyutu, döndürme açısı ve anlık imleç koordinatları.
- **Canvas Araç Çubuğu:** Görüntünün üstünde sığdırma, döndürme ve zoom butonları.
- **Klavye Kısayolları:** `F` (sığdır), `1` (gerçek boyut), `[` / `]` (döndür), `+` / `-` (zoom), `0` (sıfırla).

### 2. Birleşik Araç Mimarisi: LLM Vision Studio
Projenin en büyük kullanılabilirlik sorunlarından biri, üç ayrı script arasında geçiş yapma zorunluluğuydu: anotasyon için `annotator.py`, toplu işleme için `auto_pipeline.py` ve model eğitimi için `finetune_tool.py`. Her araçta modeller ayrı ayrı yükleniyordu, sınıf ve threshold ayarları paylaşılmıyordu, dosya yolları manuel kopyalanıyordu. Bu sorunu kökten çözmek için üç aracı tek bir tab-bazlı PyQt5 uygulamasında birleştiren **LLM Vision Studio** mimarisini tasarladım.

Uygulama dört ana tab'dan (sekmeden) oluşuyor:
- **Anotasyon Tab'ı:** Mevcut tüm araçları (9 anotasyon aracı, HITL review queue, vertex editing, TTA, undo/redo) barındırıyor.
- **Pipeline Tab'ı:** Komut satırı olan `auto_pipeline.py` scriptini görsel arayüze dönüştürüyor — ilerleme çubuğu, canlı log, istatistik kartları ve tek tıkla çalıştırma desteği.
- **Fine-Tune Tab'ı:** Eğitim aracını entegre ediyor ve `train` / `val` yollarını otomatik dolduruyor.
- **Sonuçlar Tab'ı:** Round bazlı karşılaştırma tablosu sunuyor.

Mimarinin çekirdeğinde **`SharedState`** sınıfı yer alıyor: modeller bir kez yüklenip tüm tab'larda paylaşılıyor, sınıf konfigürasyonları senkronize tutuluyor, round bazlı dosya yolları otomatik hesaplanıyor. Tab'lar arası geçiş butonları ile iş akışı kesintisiz hale getirildi: anotasyondan export edip doğrudan pipeline'a veya fine-tune'a geçebilme, fine-tune bittikten sonra modeli yükleyip anotasyona tek tıkla dönebilme imkanı sağlandı.

### 3. Arayüz Tasarım Kararları
Tab stilleri, istatistik kartları, log penceresi ve canvas araç çubuğu butonlarının renkleri ergonomik iş akışına uygun şekilde tasarlandı. `Fusion` stil motoru kullanılarak platformlar arası tutarlı bir modern görünüm sağlandı.

---

## Sonuç ve Sonraki Adımlar

Canvas iyileştirmeleri ve birleşik araç mimarisi tasarımı tamamlandı. Bu iki spesifikasyon birlikte uygulandığında, aracın kullanılabilirliği ve iş akış hızı belirgin şekilde artacak. 

Sonraki adımlarda **LLM Vision Studio** uygulamasının kodlanmasına başlanacak ve canvas iyileştirmeleri arayüze entegre edilecek.

---

## Kullanılan Teknolojiler
`Python 3.12`, `PyQt5 (QTabWidget, QGraphicsView, QPainter, sinyal/slot mimarisi)`, `Grounding DINO`, `SAM 2`, `COCO JSON formatı`

---

## Üretilen Çıktılar
- `llm_vision_studio_spec.md` — Birleşik araç mimarisi ve canvas iyileştirmeleri spesifikasyon dokümanı.

---
[Ana Sayfaya Geri Dön](../README.md) | [Gün 05 Raporu](../Day05/README.md)

---
*Hazırlayan: Durhasan*
