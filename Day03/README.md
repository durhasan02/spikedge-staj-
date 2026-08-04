# Gün 03 - Günlük Çalışma Raporu

**Tarih:** 22 Temmuz 2026  
**Konu:** Anotasyon Aracı v5 Testi, Hata Düzeltmeleri ve İyileştirmeler  

---

## Günün Özeti

Bugün bir önceki gün geliştirdiğim `annotation_tool_v5.py` aracını gerçek histoloji (H&E böbrek) görüntüleri üzerinde test ettim. Çalışma sırasında ortaya çıkan kritik bir çökme hatasını giderdim, kullanıcı deneyimini iyileştiren iki yeni özellik ekledim ve tespit kalitesini artıracak fikirleri değerlendirdim. Tüm değişiklikleri ekransız (offscreen) otomatik testlerle doğruladım.

---

## Yapılan İşler

### 1. Gerçek Görüntü Üzerinde Test ve Kritik Hata Düzeltmesi
v5 aracını glomerulus sınıfı üzerinde çalıştırırken, inceleme kuyruğundaki bir maskeyi düzenleyip onayladığımda program çöküyordu. 
- **Hatanın Kaynağı:** Maske verilerinin NumPy dizisi taşıyan sözlükler olması ve liste içi karşılaştırmaların bu dizileri `==` ile kıyaslayarak `"ValueError: The truth value of an array with more than one element is ambiguous"` hatası vermesiydi.
- **Çözüm:** Kimlik (`is`) tabanlı iki yardımcı fonksiyon yazarak tüm ekleme, silme ve karşılaştırma noktalarını düzelttim; çökme problemi tamamen ortadan kalktı.

### 2. Serbest Yakınlaştırma ve Kaydırma (Zoom / Pan)
Görüntü üzerinde istenen bölgeye hassas çalışma için canvas'a serbest zoom ve pan işlevleri eklendi:
- Fare tekerleği ile imleç merkezli yakınlaştırma.
- Orta tuş (veya `Ctrl + Sol Tık`) ile kaydırma.
- `+` / `-` ve `0` / `F` klavye kısayolları entegrasyonu.
- Tüm araçların tıklama koordinatları her zoom seviyesinde doğru eşlenecek şekilde koordinat dönüşüm matrisleri yeniden düzenlendi.

### 3. İnceleme Kuyruğu Görünürlük İyileştirmesi
Test sırasında orta/düşük güvenli maskelerin canvas'ta çok soluk ve yanıp sönerek göründüğü, kullanıcının maskeleri fark edemediği anlaşıldı:
- Maske dolguları belirginleştirildi.
- Kenarlıkların kaybolmadan yalnızca nabız gibi kalınlaşmasını sağlayan animasyon eklendi.
- Her maskenin üstüne güven skoru rozeti (`badge`) eklendi.
- Kuyruk en yüksek skordan düşüğe sıralandı; böylece en olası tespitler önce görünüyor.

### 4. Güven Bazlı İş Akışının Netleştirilmesi
Bu görüntüde DINO skorlarının düşük çıkması (`0.26 – 0.30`) nedeniyle tespitlerin otomatik kabul yerine inceleme kuyruğuna düşmesinin tasarım gereği olduğunu doğruladım. Kullanıcının maskeleri *Annotations* listesine geçirmesi için **Accept** / **Accept All Remaining** akışını ve otomatik kabul eşiğini düşürme seçeneğini netleştirdim.

### 5. Verimlilik ve Sonuç Kalitesi İçin İyileştirme Fikirleri
Düşük tespit güveninin asıl nedeninin Grounding DINO'nun histoloji görüntülerinde eğitilmemiş olması olduğunu tespit ettim. Sonuçları iyileştirmek için öncelikli fikirler belirledim:
- Küçük yapıları yakalamak için **Tiling** (parçalı tarama).
- COCO export + fine-tune ile **Aktif Öğrenme Döngüsü**.
- H&E histoloji görüntüleri için **Stain Normalization** (renk normalizasyonu).
- SAM2'de **Multimask** ile en iyi maske seçimi.
- Klavyeyle hızlı inceleme (otomatik ilerleme/onaylama akışı).

---

## v5'te Bugün Yapılan Değişiklikler Özeti

- **Hata Düzeltmesi:** İnceleme kuyruğunda NumPy dizisi kaynaklı çökme, kimlik tabanlı karşılaştırma ile giderildi.
- **Zoom / Pan:** Fare tekerleği ile imleç merkezli yakınlaştırma, orta-tuş/Ctrl+sol ile kaydırma, +/- ve 0/F kısayolları eklendi; araçlar her zoom seviyesinde doğru çalışıyor.
- **Görünürlük:** İnceleme maskeleri belirgin dolgu ve skor rozetiyle gösteriliyor, kuyruk skora göre sıralı, kenarlıklar artık kaybolmuyor.

---

## Sonuç ve Sonraki Adımlar

Gün sonunda v5 aracı gerçek görüntüler üzerinde kararlı biçimde çalışır duruma geldi; kritik çökme hatası giderildi ve kullanılabilirlik belirgin şekilde arttı. 

Sonraki adımlarda tespit kalitesini artırmak için tiling ve fine-tune tabanlı aktif öğrenme döngüsünü kurmayı, klavyeyle hızlı inceleme ve batch (klasör) işleme özelliklerini eklemeyi planlıyorum.

---

## Kullanılan Teknolojiler
`Python 3.12`, `PyTorch 2.11+cu128`, `PyQt5`, `HuggingFace Transformers (Grounding DINO, SAM 2)`, `OpenCV`, `NumPy`, `torchvision (NMS)`, `llm-vision-for-scientists (bnsreenu)`

---
*Hazırlayan: Durhasan*
