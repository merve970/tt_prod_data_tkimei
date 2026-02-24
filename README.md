# 📌 tt_prod_data_tkimei

`tt_prod_data_tkimei`, prod import dosyalarına ait kayıtları **dosya boyutu / kayıt sayısı / regex hata sayısı** ekseninde analiz eden, tek bir Jupyter notebook’undan oluşan bir veri analizi çalışmasıdır.

Notebook; `FILE_SIZE` ile `NUMBER_OF_RECORDS` ilişkisini ölçmeyi, `FILE_TP_ID` (kategori) bazında zaman serisi davranışını incelemeyi ve `REGEX_ERROR_COUNT` trendini hacim sinyali ile birlikte değerlendirerek validasyon/model yaklaşımı seçimine teknik girdi sağlamayı hedefler.

## 🚀 Özellikler

- CSV kaynağından veri yükleme, tip dönüşümleri ve temel temizlik
- Eksik veri oranları, kategori/status dağılımları ve kalite kontrolleri
- `FILE_SIZE` ↔ `NUMBER_OF_RECORDS` korelasyon analizi ve hacim sinyali seçimi
- Kategori bazında histogram/boxplot/scatter ve zaman serisi görselleştirmeleri
- Rolling metrikler ile trend/oynaklık incelemesi
- Regex hata analizi: dağılım, oranlar, hacim-hata ilişkisi ve status kırılımı
- Zaman serisi tanılama: ACF, Ljung-Box, sezonsallık testleri, STL decomposition
- Opsiyonel ileri denemeler: değişim noktası (`ruptures`), harmonik regresyon, SARIMAX denemeleri

## 🛠️ Kullanılan Teknolojiler

- Python (önerilen: 3.9+)
- Jupyter Notebook / VS Code Notebook
- Kütüphaneler: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `statsmodels`, `ruptures`

## 📦 Kurulum

### 1) Projeyi klonla

```bash
git clone https://github.com/merve970/tt_prod_data_tkimei.git
cd tt_prod_data_tkimei
```

### 2) Sanal ortam (önerilir)

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3) Bağımlılıkları yükle

```bash
pip install -U pip
pip install pandas numpy matplotlib seaborn scipy statsmodels ruptures jupyterlab nbconvert
```

### 4) Notebook konfigürasyonu

Notebook, çalıştırmadan önce birkaç değişkenin güncellenmesini bekler:

```python
# Veri yolu (notebook varsayılanı Kaggle path’idir)
CSV_PATH = "/absolute/path/to/import_file_output.csv"

# True yapılırsa yalnızca başarılı kayıtlar kullanılır
# (FILE_STATUS == "S" ve FILE_NAME .gz ile bitmeyenler)
USE_ONLY_TRAINING_DATA = False

# Hacim sinyali olarak hangi sütun kullanılacak?
VOLUME_COL = "NUMBER_OF_RECORDS"  # alternatif: "FILE_SIZE"
```

## ▶️ Çalıştırma

### Geliştirme (interaktif notebook)

```bash
jupyter lab
```

Ardından `tt-prod-data-tkimei (11).ipynb` dosyasını açın, `CSV_PATH`/dosya yolunu güncelleyin ve hücreleri sırasıyla çalıştırın.

### Production (headless çalıştırma - opsiyonel)

Notebook’u arayüz olmadan çalıştırıp çıktılı bir notebook üretmek için:

```bash
jupyter nbconvert --to notebook --execute "tt-prod-data-tkimei (11).ipynb" --output "tt-prod-data-tkimei.executed.ipynb"
```

## 📂 Proje Yapısı

```text
.
├── README.md
└── tt-prod-data-tkimei (11).ipynb
```

- `tt-prod-data-tkimei (11).ipynb`: Tüm veri hazırlama, analiz ve görselleştirme adımlarını içeren ana notebook

## 📊 Örnek Çıktılar

Notebook çalıştığında tipik olarak aşağıdaki çıktı türlerini üretir:

- Veri setinin temel özetleri (satır sayısı, eksik değer oranları, kategori/status dağılımları)
- `FILE_SIZE` ve `NUMBER_OF_RECORDS` ilişkisi için scatter/korelasyon çıktıları
- Kategori (`FILE_TP_ID`) bazında dağılım grafikleri: histogram, boxplot
- Zaman serisi çizimleri: günlük seri, `window = 30` için rolling mean/std, haftalık ortalama (`resample("W")`)
- Normalize karşılaştırmalar: kategori bazında Z-score (`VOLUME_NORM`)
- Regex hata analizi: `REGEX_ERROR_COUNT` histogramları, `HAS_REGEX_ERROR` oranları, hacim-hata scatter ve korelasyonlar
- Model seçimi için konsol çıktıları: ACF grafikleri, Ljung-Box tabloları, STL decomposition grafikleri ve özet yorumlar

Beklenen veri şeması (CSV sütunları):

```text
ID, FILE_TP_ID, FILE_NAME, FILE_STATUS, FILE_SIZE, NUMBER_OF_RECORDS,
REGEX_ERROR_COUNT, PROCESSED_DATE, CREATE_DATE
```

Not: Notebook içinde `FILE_NAME` alanından zaman bilgisi çıkarılarak `FILE_TIMESTAMP` ve `FILE_DATE` gibi türetilmiş alanlar oluşturulur.

## 🔍 Nasıl Çalışır?

1. **Yükleme & temizlik:** CSV `CSV_PATH` üzerinden okunur; sayısal alanlar (`FILE_TP_ID`, `FILE_SIZE`, `NUMBER_OF_RECORDS`, `REGEX_ERROR_COUNT`) `to_numeric(..., errors="coerce")` ile parse edilir, `FILE_STATUS` normalize edilir (`upper/strip`).
2. **Opsiyonel filtre:** `USE_ONLY_TRAINING_DATA=True` ise yalnızca başarılı kayıtlar seçilir (`FILE_STATUS == "S"` ve `.gz` ile bitmeyenler).
3. **Hacim sinyali seçimi:** `FILE_SIZE` ↔ `NUMBER_OF_RECORDS` korelasyonları kontrol edilir; `VOLUME_COL` seçilerek `VOLUME_SIGNAL` üretilir.
4. **Zaman ekseni üretimi:** `FILE_NAME` içinden `r"(\d{14})"` ile zaman damgası çıkarılır; `FILE_TIMESTAMP` ve `FILE_DATE` türetilir ve sıralama yapılır.
5. **Kategori bazlı analiz:** Histogram/scatter/boxplot; günlük seri, rolling istatistikler (`window=30`), haftalık ortalama ve Z-score normalize karşılaştırmalar.
6. **Regex hata analizi:** `HAS_REGEX_ERROR`, `ERROR_RATE` (ör. kategori 3 için) ve hacim-hata korelasyonları.
7. **Model seçimi desteği:** ACF (level ve `DELTA`), sezonsallık testleri (örn. haftalık), Ljung-Box ve STL decomposition; opsiyonel olarak değişim noktası (`ruptures`) ve SARIMAX denemeleri.

## 📌 Gereksinimler

- macOS/Linux/Windows
- Python: 3.9+ (notebook metadata: 3.12.12)
- Jupyter (Notebook/Lab) veya VS Code Notebook desteği

