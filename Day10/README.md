# Gün 10 - Günlük Çalışma Raporu

**Tarih:** 31 Temmuz 2026  
**Hazırlayan:** Durhasan Yazğan  
**Konu:** RailSem19 Video Karelerinin Elle Etiketlenmesi ve Kalıcı Dosya-Diyaloğu Hafızası  

---

## Günün Özeti

Bugün RailSem19 için bir videodan çıkardığımız kareler üzerinde **elle (manuel) ray etiketlemesi** yaptım. Etiketleme, önceki gün eklediğimiz araçların (çizgi aracı, içini doldur, SAM-P, fırça) gerçek kullanımını sağladı. Bu kullanım sırasında iş akışını en çok yavaşlatan noktalardan biri, her dosya diyaloğunun sürekli başa dönmesiydi; bunu çözmek için **kalıcı, diyalog-bazlı klasör hafızası** ekledim. Ayrıca sahada fark ettiğim iki küçük kusuru (çizgi kalınlığı ve içini-doldur taşması) düzelttim.

---

## Yapılan İşler

### 1. RailSem19 Video Karelerinin Elle Etiketlenmesi
Videodan çıkarılan kareler üzerinde ray yapılarını (rail_raised, rail_track vb.) elle etiketledim. Rayların ince ve kıvrımlı yapısı nedeniyle çizgi aracını (merkez çizgisine tıklama), iki ray arasını doldurmak için içini-doldur özelliğini ve gerektiğinde SAM-P / fırça araçlarını birlikte kullandım. Amaç, elle çizilmiş segmentasyon maskelerinden oluşan kaliteli bir YOLO-seg eğitim seti hazırlamak.

### 2. Kalıcı Dosya-Diyaloğu Klasör Hafızası (Yeni)
Etiketleme sırasında her dosya diyaloğunun aynı sabit konumdan açılması ve sürekli klasör aramak vakit kaybettiriyordu. Bunu kökten çözmek için **her diyaloğun kendi son klasörünü hatırladığı**, birbirinden bağımsız bir hafıza ekledim:
- **Anotasyon:** Load Image, Save Masks, model seçimi, Merge COCO klasörleri.
- **Studio:** Pipeline klasörleri, Fine-Tune dataset.yaml / checkpoint, YOLO11 model seçimi, Sonuçlar proje kökü.

Hafıza **QSettings** ile saklandığından uygulamayı kapatıp açsan bile korunuyor (kalıcı). Dosya seçilince üst klasör, klasör seçilince kendisi hatırlanıyor. İki ayrı süreçle (yeniden başlatma simülasyonu) test edilip doğrulandı.

### 3. Etiketleme Sırasında Fark Edilen Kusurların Düzeltilmesi
Sahadaki kullanım, 30 Temmuz'da eklediğimiz iki araçta iki kusuru ortaya çıkardı; ikisini de düzelttim:
- **Çizgi (taper) aracı:** Kalınlık slider'ı çizgi aracında gizli kaldığından çizgi hep ince çıkıyordu ve daralma yönü tıklama sırasına bağlıydı. Slider artık çizgi aracında görünür; daralma otomatik olarak görüntüdeki yakın (alt) uçtan kalın, uzak uca ince olacak şekilde yönleniyor.
- **İçini Doldur:** Önceki konveks-gövde yaklaşımı virajlarda rayların dışına taşıyordu. Satır-bazlı (raylar-arası) doldurmaya geçtim; artık dolgu eğri boyunca ray hattını takip ediyor, dışarı taşmıyor.

---

## Sonuç ve Sonraki Adımlar

Kalıcı klasör hafızası ve iki küçük düzeltme ile elle etiketleme akışı gözle görülür biçimde hızlandı. Sonraki adımlarda daha çok kare etiketlemeyi planlıyorum.

---

## Kullanılan Teknolojiler

Python 3.12, PyQt5 (QFileDialog, kalıcı ayar için QSettings), OpenCV, NumPy, SAM 2, RailSem19 video kareleri.

---

## Üretilen Çıktılar

- **`annotator.py` (güncelleme)** — Tüm dosya diyalogları için kalıcı, anahtar-bazlı klasör hafızası (QSettings); çizgi aracı kalınlık/yön düzeltmesi; içini-doldurun viraj-takipli (satır-bazlı) hâli.
- **`llm_vision_studio.py` (güncelleme)** — Studio'daki tüm dosya diyaloglarına (Pipeline, Fine-Tune, YOLO11 model, Sonuçlar) kalıcı klasör hafızası entegrasyonu.
- **Elle etiketlenen RailSem19 kareleri** — COCO (segmentation dahil) + maske çıktıları.