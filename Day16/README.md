# Gün 16 - Günlük Çalışma Raporu

**Tarih:** 10 Ağustos 2026  
**Konu:** Görüntü Anotasyon Aracına Video Desteği Planlaması ve SAM 3 / SAM 2 Video Testleri

---

## Günün Özeti

Bugün, geliştirmekte olduğumuz LLM Vision Studio aracına "Video Anotasyon" yeteneği kazandırmak için geniş çaplı araştırmalar ve testler yaptım. Tekil fotoğraflara kıyasla videoların model eğitimine daha zengin veri sağlayacağı öngörüsüyle farklı nesne takip yaklaşımlarını inceledim. Meta'nın en gelişmiş modeli olan SAM 3'ün Windows ortamına kurulumunu gerçekleştirip uyumluluk sorunlarını aşarak testlerini tamamladım. Ancak bilgisayarımızın 8 GB VRAM kısıtlaması nedeniyle araca asıl entegrasyon için daha hafif olan SAM 2 Video modelinde karar kılarak ilk bağımsız test betiklerini hazırladım.

---

## Yapılan İşler

### 1. SAM 3 Kurulumu ve Windows Entegrasyon Testleri
- **Ortam Kurulumu:** SAM 3 için izole sanal ortam (venv) oluşturulup, Hugging Face üzerinden gerekli ağırlık dosyaları indirildi.
- **Windows Uyumluluğunun Aşılması:** Orijinal kodların sadece Linux/Triton tabanlı sunucularda çalışabilme kısıtlaması, OpenCV kullanılarak yazılan özel köprü (fallback) algoritmalarıyla giderildi.
- **Test ve Çıktılar:** Modelin fotoğraflarda ve videolarda metin (prompt) komutuyla nesneleri tespit etmesi başarıyla sağlandı. Ancak SAM 3'ün yüksek VRAM kullanımı nedeniyle, video testlerinde çoklu kelime aramasından vazgeçilip tek kelimeye odaklanıldı ve video çözünürlüğü küçültülerek testler gerçekleştirildi.

### 2. Video Anotasyon Yöntem Araştırması ve Seçimi
- Mevcut yapıya zarar vermeden (eklenti mantığıyla) araca video desteği eklemek için çeşitli yöntemler kıyaslandı:
  - **SAM 2 Video:** İşaretlenen nesneyi sonraki karelere otomatik ve hızlı taşır. (8 GB VRAM sınırımızda sorunsuz çalıştığı için **Seçildi**)
  - **SAM 3:** Sadece metinle bir nesnenin tüm örneklerini bulur. (Çok ağır olduğu için ileride daha güçlü donanımlarda kullanılmak üzere **Ertelendi**)
  - **Grounded-SAM-2 & YOLO Takip:** Referans ve ileriki aşamalar (kimlik atama/sayım) için not edildi.

### 3. SAM 2 Video Denemeleri ve Geliştirmeler
- Entegrasyon öncesi bağımsız test betikleri oluşturuldu.
- Fareyle koordinat girmeye gerek kalmadan kutu (bounding box) çizme ve nokta tıklama özellikleri eklendi.
- Tren rayı gibi ince/uzun nesneler için daha uygun olan çok noktalı (multi-point) işaretleme ve takip başarıyla test edildi.

### 4. Planlama ve Sonraki Adımlar
- **Entegrasyon Planı:** Araçta yeni bir "Video" sekmesi açılacak, kare çıkarma ve takip burada yapılacak. Nesne işaretleme ve düzeltmeler mevcut anotasyon ekranında yürütülecek. Şüpheli/yanlış kareler sistem tarafından otomatik işaretlenerek kullanıcının sadece gerekli karelerde düzeltme yapması sağlanacak.
- **Sonraki Adım:** SAM 2 Video modelinin araca kodlanarak (entegrasyon) eklenmesi ve elde edilen verilerin mevcut model eğitimi hattına bağlanması.

---

## Sonuç

Mevcut aracımızın video anotasyon kabiliyetine kavuşması için gerekli yol haritası belirlenmiş ve model testleri tamamlanmıştır. SAM 3'ün Windows üzerinde çalışabilirliği ispatlansa da sistem gereksinimleri nedeniyle mevcut aracın video entegrasyon sürecinde hafif ve hızlı olan SAM 2 Video kullanılacaktır. Planlama bitmiş olup bir sonraki adımda kodlama aşamasına geçilecektir.

---
*Hazırlayan: Durhasan*
