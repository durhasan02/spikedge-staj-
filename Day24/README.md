# Gün 24 - Günlük Çalışma Raporu

**Tarih:** 20 Ağustos 2026  
**Konu:** Tool Katmanı, Ajan Döngüsü, Bilgi Tabanı (RAG) ve Arayüz

---

## Günün Özeti

Veri katmanının üstünü tamamladım. Modelin veriye eriştiği tool katmanını, tool döngüsünü çeviren ajanı, kaynaklı cevap üreten bilgi tabanını ve ikisini gösteren arayüzü yazdım.

Günün en zorlu iki başlığı beklediğim yerlerde çıkmadı: aramanın sessizce yanlış çalışması ve ücretsiz katman kotası.

---

## Yapılan İşler

### 1. Beş tool

Model veriye beş tool ile erişiyor. Dördü sensör tarafında: anlık okumalar, zaman serisi istatistikleri, alarm geçmişi ve ürün profili. Beşincisi bilgi tabanı araması.

Üç kural hepsi için geçerli. Her çıktı kendi güvenilirliğini taşıyor; yanında kalite bayrağı, sensör güveni ve verinin yaşı geliyor. Token bütçesi var; aylık 43 bin satır kovalara indirgeniyor ve 48 saatlik pencerede 3400 token yerine 440 token dönüyor. Boş sonuç hata sayılmıyor; "veri yok" mesajı dönüyor ki model eski değeri güncelmiş gibi sunmasın.

Alarm tool'unda bir hatayı gerçek bir sohbet sırasında yakaladım. Pencere filtresi alarmın başlangıcına bakıyordu. Yani 10 gün önce başlayıp hâlâ süren kritik EC arızası "son 7 gün" sorgusunda görünmüyordu. Halbuki o soruda görülmesi en gerekli şey oydu. Model haftayı "iki ana alarm" diye özetledi ve devam eden pompa arızasını atladı. Filtreyi kesişim bazlı yaptım. Devam eden alarmlar da listede öne alınıyor.

### 2. Ajan ve sistem prompt'u

İlk sürümde model EC düşüklüğünü doğru tespit ediyordu. Ama "besin takviyesi yapın" diyordu. Halbuki EC 233 saattir hedefin altında ve tek bir basamak düşüşüyle başlamış. Bu tüketim değil, dozlama pompası arızası. Takviye yapılırsa ertesi gün yine düşer.

Sistem prompt'una bir kural ekledim: sapmanın şeklinden nedene git. Ani basamak düşüşü ekipman arızasıdır. Yavaş kayma tüketimdir. Günlük salınım ortam etkisidir. Sabit değer donmuş sensördür. Ayrıca model önce neyin doğrulanması gerektiğini söylüyor, sonra müdahaleyi.

Sağlayıcı tarafında da dayanıklılık gerekti. Tek bir `503` hatası, o ana kadar yapılmış bütün tool çağrılarını çöpe atıyordu. Artık geçici hatalarda tekrar deneniyor, sonra yedek modele düşülüyor. Şunu da öğrendim: 429 ile 503 aynı şey değil. Kota limiti dakikalık pencerede olur, birkaç saniyelik bekleyiş o pencereyi kapatmaz. Ayrıca sağlayıcının önerdiği bekleme süresi varsa o kullanılıyor.

### 3. Bilgi tabanı

Yedi kaynak doküman var, toplam 476 sayfa. FAO akuaponik el kitabı, Cornell marul ve ıspanak handbook'ları, besin eksikliği rehberi, EC/pH kılavuzu ve iki kaynak listesi.

Sayfa bazlı parçalama yapmadım. FAO'nun ekinde marul girdisinin başlığı bir sayfada, talimatları diğerinde. Sayfa sınırından kesmek ikisini koparıyor ve "marul için pH kaç" sorusu cevapsız kalıyor. Doküman tek sürekli metin olarak işleniyor, sayfa numaraları ayrıca takip ediliyor. Uzun paragrafları cümle sınırında bölünce parça uzunluğu 2144 karakterden 930'a indi.

Arama hibrit çalışıyor. Gömme araması anlamsal yakınlığı iyi yakalıyor ama sayısal eşikleri kaçırabiliyor. "EC 1.4–1.8" gibi ifadelerde birebir terim araması daha isabetli. İki sonuç listesi RRF ile birleştiriliyor.

### 4. Aramada üç sessiz hata

Günün en öğretici kısmı buydu. Üçü de hata vermeden yanlış çalışıyordu.

- **Sistem tipi tercihi dışlamaya dönüşmüştü.** Sıralama önce eşleşen tipi alıyordu. FAO'nun 731 parçası hiç tükenmediği için hidroponik kaynaklar hiçbir sorguda görünmedi. Kesin ayraç yerine çarpan kullandım. Artık tercih var ama dışlama yok.
- **Aday havuzu dengesizdi.** Çarpan tek başına yetmedi. Çünkü sorun sıralamada değil havuzdaydı. 731 parçalık FAO, 115 parçalık Cornell'i havuzun dışında bırakıyordu. Doküman başına kota ekledim.
- **Tam metin araması ölüydü.** Sorgu terimleri VE ile bağlanıyordu. 6 kelimelik bir sorgu hiçbir parçada birden bulunmadığı için arama sıfır sonuç dönüyordu. Yani hibrit aramanın yarısı hiç çalışmamış. OR ile bağlayınca düzeldi.

Doğrulama olarak "pH metre kalibrasyonu" sorgusunu denedim. Hidroponik bir kaynak en üstte, uyarı etiketiyle geldi. Yani çapraz tip erişimi ve uyarı mekanizması artık gerçekten çalışıyor.

Bir de otorite sırası gerekti. FAO kendi içinde marul için hem "ideal pH 5,8–6,2" hem "6,0–7,0" diyor. Birincisi bitkinin tek başına tercihi, ikincisi akuaponik işletme aralığı. Sıra şu: ürün profili bağlayıcıdır, kaynaklar genel referanstır. Bir organizmanın optimumu sistem hedefi değildir.

### 5. Kota

Ücretsiz katmanda iki ayrı limit var ve ikisine de çarptık.

Dakikalık limitte gizli bir ayrıntı vardı. Uyumluluk katmanı 24'lük bir batch'i içeride 24 ayrı istek sayıyor. Yani batch'lemek istek sayısını azaltmıyor.

Günlük limit ise yapısal. 1062 parça var ama günlük hak 1000 istek. Kusursuz bir çalıştırma bile tek günde bitmiyor. Bu iki limit ayrı ele alınmalı. Dakikalıkta beklemek işe yarar, günlükte yaramaz. Ayrım yapmayınca beş kez bir dakika bekleyip yine aynı hatayla çıkıyordum. Çözüm ikiliydi: kapsamı çekirdek dört kaynağa daralttım ve taze kotası olan başka bir gömme modeline geçtim.

Model değiştirmenin asıl riski sessiz olması. Farklı modellerin vektörleri karşılaştırılamaz. Karışık bir indekste benzerlik hata vermeden anlamsız çıkar. Artık her parça hangi modelle gömüldüğünü kaydediyor. Uyuşmayanlar siliniyor, arama karışıklık varsa reddediyor.

Son olarak gözden kaçan bir maliyeti kapattım. Her arama da aynı günlük kotadan yiyor. Sorgu vektörleri artık önbellekte tutuluyor. Demo sorularını önceden çalıştırıp önbelleği ısıtan bir script de yazdım. Böylece sunumda kota harcanmıyor.

### 6. Arayüz

Solda canlı panel var, sağda sohbet. İkisi de aynı tool katmanını kullanıyor. Panelde sistem seçici, metrik kartları, hedef bandı gölgeli grafik ve alarm listesi bulunuyor. Sohbette tool çağrıları açılır panelde görünüyor: hangi tool, hangi parametreyle çağrılmış.

Alarm listesinde veri boşlukları varsayılan olarak kapalı. Çünkü filtresiz listede 117 alarmın 53'ü boşluk oluyor ve agronomik sinyal görünmez kalıyor.

---

## Sonuç

Chatbot uçtan uca çalışıyor. Sensör verisini okuyor, hedefle karşılaştırıyor, sapmanın şeklinden kök nedene gidiyor ve tavsiyeyi kaynak adı ve sayfa vererek söylüyor. Bilinen sınırları da açıkça yazdım. Gerçek sistemde pH'ın mutlak değeri kullanılamıyor, veri setinde 196 boşluk var ve EC ölçülmüyor, TDS'ten türetiliyor.

---
*Hazırlayan: Durhasan*
