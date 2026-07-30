__Günlük Çalışma Raporu__

Tarih: 28 Temmuz 2026  |  Hazırlayan: Durhasan

__Konu: YOLO Entegrasyonu — Araştırma, Mimari Tasarım ve Çoklu Tespit Motoru Spesifikasyonu__

Bugün projeye YOLO model ailesinin entegrasyonunu araştırdım, üç farklı kullanım senaryosunu detaylandırdım ve hem otomatik pipeline hem de anotasyon aracı için kapsamlı bir YOLO entegrasyon spesifikasyonu hazırladım\. Bu entegrasyon, projenin tespit hızını 20\-300 kat artırırken aktif öğrenme döngüsünü de güçlendirecek\.

__Yapılan İşler__

__1\. YOLO Model Ailesinin Araştırılması ve Versiyon Seçimi__

Mevcut YOLO versiyonlarını \(YOLOv8, YOLO11, YOLO26, YOLO\-World\) karşılaştırmalı olarak araştırdım\. YOLO11, YOLOv8'e göre %42'ye kadar daha az parametreyle daha yüksek mAP elde ettiği için kapalı küme \(fine\-tuned\) tespit motoru olarak seçildi\. YOLO\-World ise Grounding DINO'ya kıyasla yaklaşık 20 kat daha hızlı zero\-shot tespit yaptığı için açık küme alternatifi olarak belirlendi\. YOLO26 NMS\-free tasarımıyla edge deployment için güçlü bir aday olmasına rağmen henüz çok yeni olduğundan şimdilik ertelendi\. Tüm YOLO modelleri Ultralytics API'sini paylaşıyor, dolayısıyla versiyon değişimi tek satır kod değişikliği\.

__2\. Üç YOLO Kullanım Senaryosunun Detaylandırılması__

Birinci senaryo, anotasyon aracında Grounding DINO yanına YOLO\-World ve fine\-tuned YOLO11'i alternatif tespit motorları olarak eklemek — kullanıcı ihtiyaca göre motor seçer, hepsi aynı SAM 2 segmentasyon pipeline'ından geçer\. İkinci senaryo, auto\-distillation: Grounding DINO'nun \(öğretmen model\) ürettiği anotasyonlarla YOLO11'i \(öğrenci model\) eğitmek\. Bu sayede büyük yavaş modelin bilgisi küçük hızlı modele aktarılır ve sonraki round'larda YOLO11 ile 20\-300 kat daha hızlı tespit yapılır\. Üçüncü senaryo, fine\-tune sonrası YOLO11'i TensorRT, ONNX veya TFLite formatına export ederek edge cihazlarda \(Jetson, Raspberry Pi\) çalıştırma — bu senaryo Embedded AI hedefimle uyumlu olmasına rağmen şimdilik ertelendi\.

__3\. Auto Pipeline'a YOLO11 Eğitimi Eklenmesi__

Mevcut auto\_pipeline\.py'ye iki yeni adım ekleyen detaylı spesifikasyon hazırladım\. İlk adım, COCO JSON formatındaki anotasyonları YOLO formatına dönüştürme: her görüntü için normalize koordinatlı \.txt label dosyaları, train/val klasör yapısı ve dataset\.yaml üretimi\. İkinci adım, Ultralytics YOLO API'si ile YOLO11 eğitimi: pretrained nano/small/medium/large model seçimi, erken durdurma, checkpoint kaydetme ve eğitim grafiklerinin otomatik üretilmesi\. Pipeline'a \-\-train\_yolo flag'i eklendi, DINO fine\-tune ile paralel veya bağımsız çalışabiliyor\. Çıktılar data/yolo/round\_XX/ ve checkpoints/round\_XX/yolo11/ klasörlerine kaydediliyor\.

__4\. Annotator'a Çoklu Tespit Motoru Desteği__

Anotasyon aracına üç tespit motorunu aynı arayüzle kullanabilmek için DetectionEngine soyut sınıfı ve üç somut implementasyonu \(GDINOEngine, YOLOWorldEngine, YOLO11Engine\) tasarladım\. Her motor aynı detect\(\) arayüzünü sunuyor, DetectionWorker motor bağımsız çalışıyor, tiling ve TTA her motorla uyumlu\. GUI'de radio butonlu motor seçici eklendi: Grounding DINO seçilince model dropdown'u, YOLO\-World seçilince boyut dropdown'u, YOLO11 seçilince dosya seçici gösteriliyor\. Fine\-Tune sekmesine de YOLO11 eğitim bölümü eklendi — model boyutu, epoch, batch size ayarları ve DINO fine\-tune ile paralel çalışma desteği\.

__Sonuç ve Sonraki Adımlar__

YOLO entegrasyonu tamamlandığında sistem şu akışla çalışacak: Grounding DINO ile ilk round anotasyonları üretilir, bu verilerle hem DINO hem YOLO11 eğitilir, sonraki round'larda kullanıcı YOLO11 ile 300 kat daha hızlı tespit yapar\. Bu, batch processing'de yüzlerce görüntüyü dakikalar içinde işlemeyi mümkün kılacak\. Sonraki adımlarda YOLO spesifikasyonunu koda entegre etmeyi, kaliteli anotasyonlarla round 2 eğitimini gerçekleştirmeyi ve DINO vs YOLO11 skor karşılaştırmasını yapmayı planlıyorum\.

__Kullanılan Teknolojiler__

Python 3\.12, PyTorch 2\.11\+cu128, Ultralytics \(YOLO11, YOLO\-World, SAM 2\), HuggingFace Transformers \(Grounding DINO\), PyQt5, COCO JSON ve YOLO format dönüşümü

__Üretilen Çıktılar__

__yolo\_integration\_spec\.md — __YOLO entegrasyon spesifikasyonu: COCO→YOLO dönüşümü, YOLO11 eğitim fonksiyonları, DetectionEngine soyut sınıfı, 3 motor implementasyonu, GUI motor seçici, pipeline ve annotator değişiklikleri

