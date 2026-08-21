# Gün 22 - Günlük Çalışma Raporu

**Tarih:** 18 Ağustos 2026  
**Konu:** Hidroponik Chatbot Projesinin Mimarisi ve Tasarım Kararları

---

## Günün Özeti

Staj kapsamında ikinci projeye başladım. Bir seranın sensör verisini okuyup yetiştiricinin sorularını cevaplayan bir asistan yapacağım. Bugün kod yazmadım, mimariyi kurdum. Ana karar şu oldu: dil modeli veritabanıyla doğrudan konuşmayacak. Araya bir tool katmanı girecek.

---

## Yapılan İşler

### 1. Problem tanımı

Yetiştiricinin sorusu "marul için ideal pH nedir" değil. Sorduğu şey "benim sistemimde durum nasıl, ne yapmalıyım". Bunun cevabı üç şeyi gerektiriyor: sistemde ne okunduğu, hedefin ne olduğu ve literatürün ne dediği.

Model bunların hiçbirini bilmiyor. Bilmediğinde de susmuyor, uyduruyor. Bir sera için uydurulmuş dozaj önerisi zararsız bir hata değil.

### 2. Katmanlar

Mimariyi dört katmana ayırdım. Her katmanın tek bir sorumluluğu var.

| Katman | Sorumluluk |
|---|---|
| Veri | Ham veriyi alır, kalite etiketi basar, dakika medyanına indirger |
| Tool | Modelin veriye tek erişim yolu. Sayıyı güvenilirlik bilgisiyle döndürür |
| Bilgi tabanı | Yetiştirme kılavuzlarından kaynaklı agronomik bilgi |
| Ajan + Arayüz | Tool döngüsü, sistem prompt'u, panel ve sohbet |

Panel ile sohbet aynı tool katmanını kullanacak. Ayrı yazılsalardı zamanla ayrışırlardı. Panelde "normal" görünen bir değer sohbette "yüksek" çıkabilirdi.

Veriyi doğrudan prompt'a basmak seçenek değil. 30 günlük dakika verisi 43 bin satır. Tool-calling'de model neye ihtiyacı olduğunu kendisi söylüyor. Veri özetlenmiş geliyor, aritmetiği Python yapıyor.

### 3. Her çıktı kendi güvenilirliğini taşır

Bu, projenin bel kemiği kararı. Tool'lar çıplak sayı döndürmeyecek. Her değerin yanında kalite bayrağı, sensör güveni ve verinin yaşı gelecek.

Gerekçesi somut. Elimizdeki veri setinde pH probu kalibrasyonsuz. Model "pH 8,2" değerini tek başına görürse "asit ekle" der. Yanında "sensör güveni düşük" bilgisini görürse önce kalibrasyon isteyebilir. Model görmediği şeyi dikkate alamaz.

### 4. İki sistem

Gerçek veride ne zaman ne bozulduğu bilinmiyor. Yani tespit doğruluğu orada ölçülemez. Bu yüzden baştan iki sistem koydum. `sys-01` gerçek veri, demo burada çalışıyor. `sys-02` senaryolu sentetik veri, sadece ölçüm için. Şema ve kod ikisinde de aynı, tek fark `system_id`.

### 5. Akuaponik ile hidroponik ayrımı

Cornell marul için pH 5,8 optimum der. Hidroponikte doğru. FAO akuaponik için 6,0–7,0 der. Çünkü nitrifikasyon bakterileri asidik ortamda çalışmaz.

Aynı soruya iki doğru cevap var. Hangisinin geçerli olduğu sistemin tipine bağlı. Bu yüzden hem sistemler hem de kaynak dokümanlar tip etiketi taşıyacak.

---

## Kullanılan Teknolojiler

Zaman serisi için TimescaleDB, gömme indeksi için aynı veritabanında pgvector, kod tarafında Python. Tool-calling ve gömme için OpenAI SDK'sı üzerinden Gemini kullanılıyor. Arayüz Streamlit, veritabanı Docker Compose ile ayağa kalkıyor. Sağlayıcı `.env`'den geliyor, OpenAI'ye geçmek tek satırlık değişiklik.

---

## Sonuç

Elimde kod yok ama savunulabilir bir plan var. Dört katman, sağlayıcıdan bağımsız bir ajan, veriyle birlikte gelen güvenilirlik bilgisi ve doğruluğun ölçülebildiği ikinci bir sistem. Sıradaki adım en alttan başlamak.

---
*Hazırlayan: Durhasan*
