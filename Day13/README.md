# Gün 13 - Günlük Çalışma Raporu

**Tarih:** 5 Ağustos 2026  
**Konu:** Görüntü İşleme ve Segmentasyon Metotlarının Test Edilmesi, Karşılaştırmalı Analizi ve Railsem Veri Etiketleme Çalışmaları

---

## Günün Özeti

Bugün iki koldan ilerledim. İlk olarak, 5 farklı veri tipi (kıvrımlı ray, düz ray, düşük ışık, vb.) üzerinde 14 farklı görüntü işleme ve segmentasyon metodunu bağımsız olarak test ettim. Bu testlerin temel amacı yöntemlerin projeye olası katkılarını değerlendirmekti (henüz entegrasyon yapılmadı). İkinci olarak ise, önceki günlerde başladığım Railsem veri kümesi üzerindeki etiketleme çalışmalarıma devam ettim.

---

## Yapılan İşler

### 1. Temel Görüntü İşleme ve Filtreleme
- **Kenar/Çizgi Çıkarımı:** Canny Edge, Hough Line ve Frangi Vesselness metotları test edildi. Hough Line düz raylarda iyi sonuç verirken, Frangi Vesselness yöntemi tüp ve çizgi benzeri yapıları (raylar, zarlar) vurgulamada daha etkili oldu.
- **İyileştirme:** Bilateral Filter ile kenar koruyucu gürültü azaltma ve Gamma Correction ile karanlık (gece) sahneleri aydınlatma testleri yapıldı. Özellikle düşük ışıklı sahnelerde Gamma + CLAHE birleşimi büyük fark yarattı.
- **Özel İşlemler:** Histoloji görüntülerindeki boya varyasyonlarını gidermek için **Stain Normalization (Macenko)** yöntemi uygulandı. Otsu Eşikleme ile otomatik maske çıkarma işlemleri incelendi.

### 2. Segmentasyon Metotlarının Uygulanması
- **Klasik Yöntemler:** Hızlı bölge seçimi için Superpixel (SLIC), homojen alan seçimi için Flood Fill ve birleşik nesneleri ayırmak için Watershed algoritmaları test edildi.
- **Derin Öğrenme Tabanlı Yöntemler:** Yoğun ve bitişik hücre yapılarını (histoloji/EM) ayırmak için **StarDist** ve **Cellpose** modelleri başarıyla uygulandı. 
- Ayrıca, sıfırdan eğitilebilen bir **U-Net** segmentasyon ağı PyTorch ile kuruldu ve Otsu maskeleri pseudo-etiket olarak kullanılarak pipeline demosu gerçekleştirildi.

### 3. Çıktıların Raporlanması
- Uygulanan her metodun adım adım çıktıları `ciktilar/` klasörüne kaydedildi.
- Yan yana karşılaştırma yapılabilmesi için her yönteme ait `_panel.png` görselleri oluşturuldu. Tüm sonuçların detaylı açıklaması `RAPOR.md` ve görsel ağırlıklı `RAPOR.html` dosyalarına aktarıldı.

### 4. Railsem Veri Etiketleme Çalışmaları
- Görüntü işleme testlerine ek olarak, önceki günlerde yürütmekte olduğum **Railsem** projesi kapsamındaki veri etiketleme işlemlerime devam ettim. Frame'lerin anotasyonları planlandığı şekilde sürdürüldü.

---

## Sonuç ve Sonraki Adımlar

Görüntü işleme tarafında uygulanan hiçbir metot henüz ana projeye entegre edilmedi; tamamen bağımsız bir kullanılabilirlik testi (Proof of Concept) olarak gerçekleştirildi. Ancak yapılan testler sonucunda farklı görüntü tipleri için potansiyel yaklaşımlar belirlendi (örneğin raylar için Frangi Vesselness, karanlık ortamlar için Gamma Correction). 

Sonraki adımlarda, bu testlerden elde ettiğim çıktılara dayanarak projeye değer katacağı kesinleşen yöntemleri seçip anotasyon aracımıza entegre etmeyi değerlendireceğiz. Eş zamanlı olarak Railsem etiketleme sürecine de devam edeceğim.

---

## Kullanılan Teknolojiler
`Python 3.12`, `OpenCV`, `NumPy`, `SciPy`, `scikit-image`, `PyTorch (U-Net)`, `TensorFlow (StarDist)`, `Cellpose`

---
*Hazırlayan: Durhasan*
