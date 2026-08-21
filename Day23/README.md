# Gün 23 - Günlük Çalışma Raporu

**Tarih:** 24 Ağustos 2026  
**Konu:** Veri Katmanı, Kalite Hattı ve Alarm Türetme

---

## Günün Özeti

Mimarinin en alt katmanını kurdum. Gerçek bir akuaponik veri setini sorgulanabilir ve kalite etiketli bir veritabanına dönüştürdüm. Üzerine alarm türetmeyi ekledim. Ayrıca doğruluğu ölçebilmek için senaryolu sentetik bir ikinci sistem ürettim.

Günün büyük kısmı veriyi temizlemekle geçmedi. Verinin neye benzediğini anlamakla geçti. İlk bakışta doğru görünen birkaç varsayım yanlış çıktı.

---

## Yapılan İşler

### 1. Ham satır analiz birimi değil

Veri seti 26 Ocak – 22 Mart 2023 arası 505.730 satır. İlk bakışta tekrarlı görünüyor. Zaman damgaları dakikaya yuvarlanmış ve her dakikaya ortalama 10,5 satır düşüyor.

Standart refleks tekrarları atmak olurdu. Bu, verinin %90'ını atardı. Çünkü bunlar tekrar değil, dakika içi ayrı örnekler. Aralarındaki fark da küçük değil: dakika içi pH aralığı ortalama 1,47 birim. Hiçbir çözelti saniyeler içinde 1,5 pH birimi oynamaz. Bu, filtrelenmemiş prob gürültüsü.

Analiz birimini dakika medyanı yaptım. Ortalama değil, çünkü medyan spike'a dayanıklı. Dakika içi yayılımı ve örnek sayısını da sakladım. Kalite kuralları bunları kullanıyor.

Aynı inceleme pH kanalının güvenilmez olduğunu gösterdi. Günlük pH medyanı 55 gün boyunca 5,5 ile 11,7 arasında geziyor. Fiziksel bir sistem böyle davranmaz, prob kalibrasyonsuz. TDS ve su sıcaklığı ise temiz. Bunu satır bazlı bir bayrak anlatamaz. Kanal bazlı bir ifade gerekti: `sys-01` pH için sensör güveni "düşük".

### 2. İki aşamalı kalite hattı

Kalite kontrolünü iki seviyeye ayırdım. Örnek seviyesinde fiziksel sınır dışı değerler ve elektriksel spike'lar reddediliyor. Dakika seviyesinde makul aralık dışı değerler `bad` oluyor. Ani sıçrama, donmuş sensör ve aşırı yayılım ise `suspect`.

Kritik ayrıntı spike eşiğinin yerel olması. Başta global bir eşik kullandım. Ama TDS 55 günde 267'den 538 ppm'e kayıyor. Global eşik sağlıklı taban değerleri outlier sanıp %12 yanlış red üretti. Yerele geçince 1,52 milyon örnekten sadece 26.237'si reddedildi.

Sonuçlar tutarlı çıktı. Su sıcaklığında 1.788 dakika donmuş bulundu. Hepsi sıcaklığın aktif değiştiği saatlerde. Yani gece kararlılığı değil, gerçek sensör takılması.

### 3. Şema ve zaman kaydırması

Veri TimescaleDB'de tutuluyor. Chatbot dakika medyanlarını okuyor. Ham örnekler denetim izi olarak ayrı duruyor. Saatlik ve günlük özetler üstüne kurulu.

Veri 2023 tarihli. Bu yüzden "son 24 saat" sorguları boş dönüyordu. Son okumayı "şimdi"ye kaydıran bir mekanizma ekledim. Değerler değişmiyor, sadece zaman ekseni kayıyor. Orijinal zaman ayrı kolonda duruyor.

Kaydırma yükleme anında sabitleniyor. Sonra veri gerçek zamanla yaşlanıyor. Birkaç saat sonra her cevap "bu okuma 289 dakika eski" uyarısıyla açılıyordu. Yeniden sabitlemek için ayrı bir script yazdım. 1,7 milyon satırı 15 saniyede kaydırıyor.

Bunu düz bir `UPDATE` ile yapmak mümkün değildi. TimescaleDB'de zaman kolonunu güncellemek satırı başka bir chunk'a taşıyor ve kısıta takılıyor.

### 4. Alarm türetme

Alarmları üç türe ayırdım. Agronomik sapmada veri sağlam, sistem yanlış çalışıyor. Sensör arızasında veri güvenilmez. Bir de veri boşluğu var. Bunlar karışırsa chatbot soğutma arızasını sensör hatası sanar.

Ham hâliyle üreteç kullanılamazdı. `sys-01`'de 296 alarm çıkıyordu. Çoğu sınıra teğet gezen küçük sapmalardı. Üç mekanizma ekledim. Ölü bant, hedefin %5'inden az aşımları saymıyor. Boşluk katlama, gateway koptuğunda susan metrikleri tek alarma indiriyor. Üçüncüsü de aynı ölçümün iki adı olan EC ve TDS'ten birini bastırıyor.

En öğretici düzeltme başkaydı. Kalite hattından geçen ama yine de yanlış olan pH değerleri hedef alarmı tetikliyordu. pH'ın 65 alarmının 52'si agronomikti. Bu kendi içinde çelişkiydi. Sensör güveni "bu değere dayanma" derken üreteç tam o değerden 52 sonuç çıkarıyordu. Artık güveni düşük kanallardan agronomik alarm üretilmiyor. Yerine tek bir meta-alarm kalıyor. Kaç olayın bastırıldığını söylüyor ve önce kalibrasyon istiyor. Toplam 171 alarm 117'ye indi, pH 65'ten 14'e.

### 5. Ölçüm için sentetik sistem

Gerçek veride ne zaman ne bozulduğu bilinmiyor. `sys-02` bunun için. İçine zaman damgalı altı arıza enjekte ettim: donmuş pH probu, dozlama pompası arızası, soğutma arızası, oksijen düşüşü, rastgele spike'lar ve altı saatlik veri boşluğu. Beklenen sonuçlar dosyada etiketli.

Üreteç `sys-02`'de tam 6 alarm çıkarıyor. Altısı da enjekte edilen senaryolara birebir denk geliyor. Yanlış pozitif yok. Donmuş prob senaryosunda 480 dakikanın 479'u yakalandı.

Bunun için sentetik profili de düzeltmek gerekti. Ortam metrikleri arıza yokken bile bant dışındaydı ve eval'i kirletiyordu. Oksijen senaryosu ise hiç tetiklenmiyordu. Sıcaklık artışı tek başına yeterli düşüş yaratmıyor. Artan biyolojik oksijen talebini de modelleyince senaryo test edilebilir hale geldi.

---

## Sonuç

Veri katmanı hazır. Sorgulanabilir, kalite etiketli ve kanal bazlı güven bilgisi taşıyor. Üzerinde üç türe ayrılmış bir alarm tablosu var. Doğruluğun ölçülebildiği sentetik sistem de kuruldu ve ilk ölçüm temiz geçti. Sıradaki adım bu katmanı modele açmak.

---
*Hazırlayan: Durhasan*
