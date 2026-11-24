# End-to-End Data Lakehouse & Machine Learning Pipeline: Prediksi Harga Properti Melbourne

Proyek ini merupakan implementasi arsitektur **Data Lakehouse** secara *end-to-end* menggunakan platform **Databricks** untuk memenuhi **Tugas 2 Mata Kuliah Big Data**. Sistem ini mencakup proses *incremental data ingestion*, transformasi *Medallion Architecture* (*Bronze–Silver–Gold*), orkestrasi otomatis berbasis *File Arrival Trigger*, pelatihan model *Machine Learning* (Regresi), *deployment* REST API, pengujian melalui Postman, hingga visualisasi *AI/BI Dashboard*.

---

| Azzam Abdurrohman | 2311103018 | S1SI07REG01 |

---

## Deskripsi Dataset & Strategi Batch

Dataset yang digunakan adalah **Melbourne Housing Snapshot** (*open access* dari Kaggle) yang berisi **13.580 rekaman transaksi properti** dan **21 atribut** di Melbourne, Australia. 

Untuk mensimulasikan skenario aliran data bertahap (*incremental ingestion*) pada arsitektur Lakehouse, dataset dibagi menjadi dua *batch* tanpa mengubah kondisi data mentahnya (*raw*):

| Nama File | Jumlah Baris | Proporsi | Peran dalam Pipeline |
| :--- | :---: | :---: | :--- |
| `melb_batch1.csv` | 9.500 | ~70% | *Ingestion* awal untuk inisialisasi tabel *Bronze*, *Silver*, *Gold*, dan pelatihan model awal. |
| `melb_batch2.csv` | 4.080 | ~30% | Simulasi kedatangan file baru untuk memicu *Workflow Job* secara otomatis (*File Arrival Trigger*). |
| **Total** | **13.580** | **100%** | **Keseluruhan data mentah pada lapisan Bronze.** |

### Karakteristik Utama Data
* **Target Prediksi (Regresi)**: `Price` (harga jual properti dalam AUD).
* **Fitur Numerik**: `Rooms`, `Distance`, `Bedroom2`, `Bathroom`, `Car`, `Landsize`, `Lattitude`, `Longtitude`, `Propertycount`.
* **Fitur Kategorikal**: `Type` (`h`: *house*, `u`: *unit*, `t`: *townhouse*), `Method`, `Regionname`, `CouncilArea`, `Suburb`.
* **Kualitas Data (*Data Quality*)**: Dataset memiliki nilai kosong (*missing values*) pada kolom `BuildingArea`, `YearBuilt`, `CouncilArea`, dan `Car` yang ditangani secara otomatis pada lapisan **Silver**.

---

## Arsitektur Sistem (End-to-End)

```text
[GitHub CSV Files] 
       │
       ▼
[Unity Catalog Volume: /Volumes/workspace/melbourne_housing/raw_data/incoming/]
       │
       ▼ (Auto Loader + File Arrival Trigger)
[Lakeflow ETL Pipeline - Medallion Architecture]
  ├── Bronze Layer : bronze_melbourne (13.580 baris data mentah + metadata)
  ├── Silver Layer : silver_melbourne (12.211 baris bersih & tervalidasi)
  └── Gold Layer   : gold_melbourne   (Agregasi rata-rata & total harga per wilayah/tipe)
       │                                      │
       ▼                                      ▼
[AI/BI Dashboard]                    [MLflow & Scikit-Learn]
(4 KPI & 4 Visualisasi)              (RandomForestRegressor)
                                              │
                                              ▼
                                     [Model Serving Endpoint] ──► [Uji API via Postman]
