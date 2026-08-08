# Gün 14 - Günlük Çalışma Raporu

**Tarih:** 6 Ağustos 2026  
**Konu:** Railsem Veri Etiketleme Çalışmalarının Tamamlanması ve Görüntü İşleme Entegrasyonuna Başlanması

---

## Günün Özeti

Bugün öncelikle Railsem projesi kapsamında 100 adet frame etiketleyerek bu veri setindeki etiketleme görevimi tamamen bitirdim. Ardından, `annotator.py` aracına yeni görüntü işleme özelliklerinin entegrasyonu çalışmalarına başladım. Mevcut mimariyi koruyarak özelliklerin temel bir kısmını başarıyla ekledim.

---

## Yapılan İşler

### 1. Railsem Veri Etiketleme
- Railsem veri kümesi için planlanan son **100 adet frame'in** etiketleme işlemi tamamlandı.
- Bu proje kapsamındaki etiketleme görevleri başarıyla sonlandırıldı.

### 2. Görüntü İşleme Entegrasyonuna Giriş (`annotator.py`)
`annotator.py` anotasyon aracına eklenmesi planlanan 7 yeni görüntü-işleme özelliğinin altyapısı kuruldu ve ilk modülleri entegre edildi.

#### Kapsam ve Temel Kurallar
- Mevcut tüm özellikler korundu (10 araç, HITL review, TTA, undo/redo, vertex editing, vb.).
- **Orijinal görüntü dosyası asla değiştirilmez** — tüm filtreler yalnız görüntüleme/ön-işleme amaçlıdır.
- Performans için filtreler cache'lenir (parametre değişmezse yeniden hesaplanmaz).
- Filtre zinciri sırası: **Bilateral → Gamma → CLAHE → Display** olarak belirlendi.
- Overlay'ler (Canny, Superpixel sınırları) filtrelerden bağımsız, üst katmanda çizildi.

#### Eklenen İlk Özellikler
1. **Morfoloji (Maske İşlemleri):** Seçili maske(ler)e morfolojik işlemler (Dilate, Erode, Temizle, Boşlukları Doldur) uygular. Anotasyon listesinden seçili satırlara çalışır ve geri alınabilir (undo destekli).
2. **Flood Fill (Magic Wand - `W`):** Tıklanan piksele benzer renkteki komşu alanı seçer. Tolerans ayarı ve mikro-undo (Shift+tık) destekler.
3. **Canny Kenar Overlay (`Ctrl+J`):** Kenar haritası yeşil yarı-şeffaf overlay olarak görüntünün üstünde çizilir.
4. **SLIC Superpixel (`X`):** Sınırları sarı overlay olarak gösterir ve SPx fırçası ile hızlı bölge seçimi sağlar.
5. **Filtre Zinciri:** Gamma (`Ctrl+G`), Bilateral (`Ctrl+K`), ve CLAHE (`Ctrl+H`) için altyapı oluşturularak arayüze bağlandı.

---

## Sonraki Adımlar
- Görüntü işleme entegrasyonunun (filtrelerin tam davranışları, tespit öncesi işleme, panel düzenlemeleri) tamamlanması.
- Uygulanan özelliklerin test edilmesi ve uçtan uca doğrulanması.

---
*Hazırlayan: Durhasan*
