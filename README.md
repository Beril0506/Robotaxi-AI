# Robotaxi-AI

YOLOv8n tabanlı nesne tespiti, şerit tespiti ve yol segmentasyonu.

## İçerik

| Dosya | Açıklama |
|---|---|
| `object-det.py` | YOLOv8n tabanlı 19 sınıflı nesne tespiti. LiDAR (`/scan`) ile mesafe füzyonu içerir. |
| `lanev2.py` | Şerit tespiti |
| `best.pt` | Eğitilmiş YOLOv8n model ağırlıkları |
| `results/` | Eğitim sonuçları (grafikler, karışıklık matrisi, örnek tahminler) |

## Tespit Edilen Sınıflar (19 sınıf)

| ID | Sınıf | ID | Sınıf |
|---|---|---|---|
| 0 | keep_left | 10 | no_right_turn |
| 1 | keep_right | 11 | park |
| 2 | no_entry | 12 | pedestrian |
| 3 | no_parking | 13 | red_light |
| 4 | crosswalk_sign | 14 | right |
| 5 | crosswalk | 15 | roundabout |
| 6 | go_ahead | 16 | stop_sign |
| 7 | green_light | 17 | tunnel |
| 8 | left | 18 | yellow_light |
| 9 | no_left_turn | | |

## Model Eğitim Sonuçları

### `results/results.png`
Eğitim/doğrulama kayıpları ve precision/recall/mAP metriklerinin epoch
bazında değişimi.
Model 50 epoch eğitilmiştir. Eğitim ve doğrulama kayıpları (box, cls, dfl) boyunca düzenli olarak düşmüş, doğrulama kaybında artış görülmediği için belirgin bir aşırı öğrenme (overfitting) gözlenmemiştir.
mAP@0.5 eğitimin sonuna doğru yükselmeye devam etmiş ve son epoch'larda yaklaşık 0.83 seviyesine ulaşmıştır (en iyi değer eğitimin son epoch'larında elde edilmiştir). Bu aşamada mAP@0.5:0.95 yaklaşık 0.65, precision yaklaşık 0.87, recall yaklaşık 0.8'dir.

### `results/confusion_matrix_normalized.png`
Sınıf bazlı normalize edilmiş karışıklık matrisi.
En çok karışan sınıflar no-left-turn-sign ve no-right-turn: iki levha yalnızca ok yönüyle ayrıldığı için model birbirine karıştırıyor (no-right-turn'ün %23'ü no-left-turn-sign, no-left-turn-sign'ın %11'i no-right-turn olarak tahmin edilmiş).
Trafik ışıkları (red, green, yellow) ve roundabout için arka plan ile karışma yüksek: bu sınıflar görüntüde küçük yer kapladığı için bazı örnekler kaçırılıyor ve bazı arka plan bölgeleri ışık olarak algılanıyor (green %43, red %29 yanlış pozitif).


### `results/BoxPR_curve.png`
Precision-Recall eğrisi.
Tüm sınıflar için mAP@0.5 = 0.834.
Keep-Left, Keep-Right, go ahead, right, stop ve tunnel sınıflarında AP 0.99 ve üzerinde; en düşük AP değerleri roundabout (0.595), no-right-turn (0.596) ve no-left-turn-sign (0.615).
Güven eşiği 0.5 olarak belirlenmiştir (bkz. KTR 7.1.3.4)

### `results/val_batch0_pred.jpg`
Doğrulama setinden örnek model tahminleri.

## Veri Seti
Veri seti Roboflow üzerinde oluşturulmuş olup araç hazır verisetlerinin düzenlenip birleştirilmesiyle oluşmaktadır.
-Toplam: 26.094 görüntü (Eğitim:23.430 / Doğrulama 1.663 / Test: 1,001)
-Ön İşleme: 640x640 boyutuna yeniden boyutlandırma
-Augmentasyon: rotasyon ±15°, parlaklık ±%25, pozlama ±%15, bulanıklık en fazla 1.6 px
Sınıflar (19): Keep-Left, Keep-Right, No Entry, No parking, Yaya Gecidi Levhasi, crosswalk, go ahead, green, left, no-left-turn-sign, no-right-turn, park, pedestrian, red, right, roundabout, stop, tunnel, yellow.


## Kullanım

```bash
python object-det.py --source <görüntü_veya_video_yolu>
ros2 run robotaksi_perception segmentation_node
```

## Bağımlılıklar

```bash
pip install ultralytics opencv-python
```
