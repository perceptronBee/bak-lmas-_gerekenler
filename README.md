# 🚀 Gelişmiş Bilgisayarlı Görü Veri Boru Hattı (Data Pipeline)

## 📌 Genel Bakış
Bu doküman, modelimizin eğitiminde kullanılacak verinin toplanması, analiz edilmesi, işlenmesi ve modele beslenmesi süreçlerindeki mühendislik standartlarımızı belirler. Amacımız; veri kalitesini maksimize etmek, donanım kaynaklarını (GPU/CPU) en verimli şekilde kullanmak ve model doğruluğunu akademik (SOTA) seviyelere çıkarmaktır.

---

## 🗂 1. Veri Kaynakları ve Toplama (Sourcing)
Sıfırdan veri toplamak yerine, öncelikle literatürdeki ve açık kaynaklardaki veri setleri taranacaktır.
* **Kullanılacak Platformlar:**
  * [Roboflow Universe](https://universe.roboflow.com/): Hazır etiketli nesne tespiti verileri için.
  * [PapersWithCode](https://paperswithcode.com/datasets): Güncel akademik makalelerin kullandığı benchmark veri setleri.
  * [Hugging Face Datasets](https://huggingface.co/datasets): Topluluk destekli geniş veri havuzu.
  * [Google Dataset Search](https://datasetsearch.research.google.com/): Genel veri taramaları için.
* **Not:** Kullanılan veri setlerinin lisans durumlarına (ticari/akademik kullanım izni) dikkat edilmelidir.

---

## 📊 2. Keşifsel Veri Analizi (EDA & Veri Temizliği)
Veriyi modele vermeden önce karakteristiği analiz edilmeli ve olası darboğazlar tespit edilmelidir.
* **Sınıf Dengesi (Class Imbalance):** Azınlık ve çoğunluk sınıflarının dağılımı kontrol edilecek.
* **Çözünürlük ve En-Boy Oranı:** Görüntülerin ve *Bounding Box*'ların boyut dağılımları analiz edilecek (Gerekirse Anchor Box optimizasyonu için K-Means uygulanacak).
* **Etiket Gürültüsü (Label Noise):** Yanlış etiketlenmiş verilerin tespiti için [Cleanlab](https://github.com/cleanlab/cleanlab) veya benzeri araçlar kullanılarak veri seti temizlenecek.

---

## ⚡ 3. Donanım ve I/O Optimizasyonu
GPU'nun veri beklerken boşta kalmasını (GPU starvation) engellemek için veri yükleme adımları optimize edilecektir.
* **Dataloader Ayarları:** CPU çekirdek sayısına uygun `num_workers` değeri belirlenecek ve RAM-VRAM transferini hızlandırmak için `pin_memory=True` kullanılacak.
* **Veri Formatı:** Gerekli durumlarda on binlerce ham görüntü dosyasını tek tek okumak yerine; ardışık (sequential) okuma yapabilen **LMDB**, **TFRecord** veya **WebDataset** formatlarına geçiş yapılacaktır.

---

## 🎨 4. Veri Zenginleştirme (Data Augmentation)
Modelin ezberlemesini (overfitting) önlemek ve genelleme yeteneğini artırmak için endüstri standardı araçlar kullanılacaktır.
* **Kütüphane:** PyTorch'un varsayılan modülleri yerine, daha hızlı çalışan ve Bounding Box'ları otomatik güncelleyen **Albumentations** kütüphanesi tercih edilecektir.
* **Stratejiler:**
  * **Piksel Seviyesi:** Hafif Motion Blur, aydınlatma değişimleri, renk sapmaları.
  * **Uzamsal Seviye:** Rastgele kırpma, döndürme.
  * **Karma (Mix) Teknikler:** Özellikle nesne tespiti performansını artırmak için **Mosaic** ve **CutMix** augmentasyonları veri hattına entegre edilecektir.

---

## 🚀 5. İleri Seviye Stratejiler (Faz 2)
Temel (Baseline) model eğitildikten sonra, model başarımını maksimize etmek için aşağıdaki yöntemler uygulanacaktır.

* **Test-Time Augmentation (TTA):** Çıkarım (inference) aşamasında, görüntünün farklı varyasyonlarının tahmin ortalaması alınarak doğruluk artırılacak.
* **Pseudo-Labeling & Self-Training:** 1. Başarılı olan temel model (Teacher) ile etiketsiz devasa veri setleri üzerinde tahmin yapılacak.
  2. Yüksek güven skoruna (Confidence > %90) sahip tahminler yalancı etiket (pseudo-label) olarak kabul edilecek.
  3. Yeni ve daha büyük veri setiyle nihai model (Student) sıfırdan eğitilecek.

---

## ✅ Aksiyon Planı ve Görevler
- [ ] Uygun açık kaynak veri setlerinin bulunması ve indirilmesi.
- [ ] EDA scriptlerinin yazılması (Sınıf dağılımı ve BBox analizi).
- [ ] Albumentations tabanlı veri zenginleştirme pipeline'ının kodlanması.
- [ ] Dataloader optimizasyonlarının donanıma göre ayarlanması.
