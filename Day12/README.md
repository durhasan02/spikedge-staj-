# Gün 12 - Günlük Çalışma Raporu

**Tarih:** 4 Ağustos 2026  
**Konu:** Görüntü İşleme Yöntemleri Araştırması ve Frame Etiketleme  

---

## Günün Özeti

Bugün anotasyon aracına entegre edilebilecek klasik görüntü işleme yöntemlerini kapsamlı şekilde araştırıp değerlendirdim ve tren rayı frame etiketleme çalışmasına devam ettim.

---

## Yapılan İşler

### 1. Görüntü İşleme Yöntemleri Araştırması
Anotasyon aracına entegre edilebilecek görüntü işleme yöntemlerini dört kategoride araştırdım: ön-işleme/iyileştirme, kenar/özellik çıkarma, segmentasyon yardımcıları ve morfolojik işlemler. Toplam 15 yöntemi uygunluk, işlevsellik ve entegrasyon zorluğu açısından değerlendirdim. Araştırma sonucunda en yüksek etkili 9 yöntem belirlendi.

Önerilen yöntemlerden öne çıkanlar: 
- **Bilateral Filter:** Kenar koruyarak gürültü azaltma, SAM maske kalitesini artırır.
- **Canny Edge overlay:** Kenarları yarı-şeffaf katman olarak göstererek etiketleme hassasiyetini artırma.
- **Superpixel SLIC:** Benzer pikselleri bölgelere ayırıp tek tıkla seçme, etiketleme hızını katlar.
- **GrabCut:** GPU gerektirmeyen SAM alternatifi segmentasyon.
- **Gamma Correction:** Parlaklık ayarı slider'ı.
- **Hough Line Transform:** Tren raylarındaki düz çizgileri otomatik bulma.
- **Watershed:** Birbirine dokunan nesneleri ayırma.

*Histogram equalization, Gabor filter ve active contours gibi yöntemler ise mevcut araçlarımızın (CLAHE, SAM 2) zaten daha iyi yaptığı işler olduğundan elenmiştir.*

### 2. Frame Etiketleme
Tren rayı projesi kapsamında video frame'lerini etiketlemeye devam ettim. LLM Vision Studio'daki anotasyon araçlarını (Poly, Brush, Çizgi) kullanarak frame'leri üç sınıfta (`rail-raised`, `rail-track`, `trackbed`) etiketledim ve COCO JSON + PNG mask olarak export ettim.

---

## Sonuç ve Sonraki Adımlar

Görüntü işleme araştırması tamamlandı ve entegrasyon öncelikleri belirlendi. Sonraki adımlarda Canny edge overlay, bilateral filter ve superpixel fırça gibi yüksek etkili yöntemleri araca entegre etmeyi ve etiketleme çalışmasına devam etmeyi planlıyorum.

---

## Kullanılan Teknolojiler
`OpenCV (Canny, Hough, SLIC, GrabCut, Watershed, bilateral filter, morfoloji)`, `NumPy`, `LLM Vision Studio`


---
*Hazırlayan: Durhasan*
