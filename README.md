# tt_prod_data_tkimei

# PROD Data Analizi

Bu proje, `tt-prod-data-tkimei (11).ipynb` notebook’u içinde prod import dosyalarına ait kayıtların **dosya boyutu / kayıt sayısı / hata sayısı** ekseninde analiz edilmesi için hazırlanmıştır.

Notebook’un amacı:
- `FILE_SIZE` ve `NUMBER_OF_RECORDS` ilişkisini incelemek,
- kategori (`FILE_TP_ID`) bazında zaman serisi davranışını analiz etmek,
- `REGEX_ERROR_COUNT` trendini ve dosya hacmi ile ilişkisini değerlendirmek,
- analiz sonuçlarına göre validasyon/model yaklaşımını seçmeye yardımcı olmaktır.

## Dosya Yapısı

- `tt-prod-data-tkimei (11).ipynb`: Tüm analiz adımlarını içeren ana notebook.

## Beklenen Veri Formatı

Notebook, CSV veri kaynağında aşağıdaki alanları bekler:

- `ID`
- `FILE_TP_ID`
- `FILE_NAME`
- `FILE_STATUS`
- `FILE_SIZE`
- `NUMBER_OF_RECORDS`
- `REGEX_ERROR_COUNT`
- `PROCESSED_DATE`
- `CREATE_DATE`

Not: Notebook içinde `FILE_NAME` alanından zaman damgası çıkarılarak `FILE_TIMESTAMP` ve `FILE_DATE` türetilir.

## Analiz Akışı (Özet)

1. **Veri yükleme ve tip dönüşümleri**
   - CSV okunur, sayısal sütunlar temizlenir ve parse edilir.
   - `FILE_STATUS` normalize edilir (`upper`, `strip`).

2. **Kalite kontrolleri**
   - Eksik veri oranları,
   - Kategori ve status dağılımları,
   - başarılı kayıt filtreleme opsiyonu (`USE_ONLY_TRAINING_DATA`).

3. **Hacim sinyali seçimi**
   - `FILE_SIZE` ve `NUMBER_OF_RECORDS` korelasyonu incelenir.
   - `VOLUME_COL` seçilir ve `VOLUME_SIGNAL` üretilir.

4. **Kategori bazlı trend ve dağılım analizi**
   - Histogram, boxplot, scatter,
   - zaman serisi çizimleri,
   - rolling ortalama / std,
   - normalize edilmiş karşılaştırmalar.

5. **Regex hata analizi**
   - `REGEX_ERROR_COUNT` dağılımı,
   - kategori bazlı hata oranı (`HAS_REGEX_ERROR`),
   - hacim-hata korelasyonları,
   - status ile hata ilişkisi.

6. **Model seçimi için detaylı zaman serisi testleri**
   - ACF, Ljung-Box,
   - haftalık/aylık/çeyreklik sezonsallık testleri,
   - STL decomposition,
   - Median/MAD tabanlı oynaklık ve outlier incelemesi,
   - ek olarak `ruptures`, harmonik regresyon (sin/cos), SARIMAX denemeleri.

## Gereksinimler

Python 3.9+ önerilir.

Temel paketler:

```bash
pip install pandas numpy matplotlib seaborn scipy statsmodels ruptures
```

## Çalıştırma

1. Notebook’u Jupyter veya VS Code Notebook ortamında açın.
2. Veri dosya yolunu güncelleyin:
   - Mevcut örnek: `/kaggle/input/datasets/aycaatac/prod-data/import_file_output.csv`
   - Kendi ortamınızda uygun `CSV_PATH` değerini verin.
3. Hücreleri sırayla çalıştırın.

## Notlar

- Notebook keşif/analiz odaklıdır; üretim pipeline’ı değildir.
- Sonuçların güvenilirliği veri kalitesine, tarih parse doğruluğuna ve kategori başına örnek sayısına bağlıdır.
- Model seçimi bölümü, tek bir kesin model dayatmak yerine veri davranışına göre yöntem seçimini destekler.
