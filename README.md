# Web-Scraping-Real-Estate-Price-Prediction

## 1. Overview
This is an end-to-end data project that:
1) crawls Vietnamese real-estate listings from the web,  
2) cleans + engineers features from messy scraped attributes (especially **price**), and  
3) trains regression models to **predict house prices**.

The repository is organized as a clear pipeline from **URL construction → crawling → preprocessing → modeling**.


## 2. Repository Structure (Notebooks)
> The workflow is implemented through these notebooks (run in order):

1. **`Create URL Tails, City Dictionary and District Dictionary.ipynb`**  
   - Builds URL tails and mapping dictionaries (City/Province and District) to systematically generate crawl targets.

2. **`Crawl Data.ipynb`**  
   - Crawls listing pages and detail pages, parses HTML, and exports the raw dataset.

3. **`Data Preprocessing and Feature Engineering.ipynb`**  
   - Cleans scraped data (especially price fields), standardizes units, handles missing values, and creates model-ready features.

4. **`Methodology - Model.ipynb`**  
   - Trains and evaluates multiple regression models; includes PCA-based analysis and model comparison.


## 3. Data Source
- Target website (example base URL used in the notebooks):  
  `https://batdongsan.vn/ban-nha/`

**What gets collected**
- Listing-level details (ID, address, location)
- Property specs (area, dimensions, floor count, room counts)
- Amenities / boolean attributes (e.g., balcony, garden, nearby school/hospital/market/mall)
- Legal & ownership flags (e.g., pink book / legal completion)
- Time information (posting date)
- Price information (from multiple sources on the page)
- URL reference to each listing


## 4. Dataset Description

### 4.1 Raw Output (Crawled Data)
The crawl produces a raw table that typically includes these groups of columns:

#### A) Identification & Location
- `Mã tin` (Listing ID)
- `Địa chỉ` (Address)
- `Quận/Huyện` (District)
- `Tỉnh/Thành phố` (City/Province)
- `Kinh độ`, `Vĩ độ` (Longitude/Latitude)

#### B) Property Specifications
- `Chiều dài`, `Chiều rộng` (Length/Width)
- `Diện tích` (Area)
- `Tầng` (Floors)
- `Phòng ngủ`, `Phòng WC`, `Phòng khách`, `Nhà bếp` (Rooms)

#### C) Amenities / Surroundings (Binary Features)
Examples:
- `Sân thượng/Ban công`
- `Sân vườn/Sân trước/Sân sau`
- `Nhà mặt tiền`
- `Gần trường học`, `Gần bệnh viện`, `Gần chợ`, `Gần siêu thị/trung tâm thương mại`

#### D) Legal / Ownership Flags (Binary Features)
Examples:
- `Sổ hồng`
- `Chính chủ`
- `Pháp lý chuẩn`
- `Hoàn công đủ`

#### E) Time & Price (Raw)
- `Ngày đăng`
- `Giá từ thẻ HTML` (Price extracted from HTML tag)
- `Giá từ mô tả` (Price mentioned in description)
- `Giá theo mét vuông từ mô tả` (Price per m² from description)
- `Url`


## 5. Project Stages (End-to-End Pipeline)

## Stage 1 — URL Engineering & Dictionaries

### Goal
Create a systematic way to crawl across **all provinces/cities** and **districts** by generating URL tails and mapping them to human-readable names.

### Methods Used
- HTML parsing utilities (e.g., `BeautifulSoup`)
- URL tail generation strategy
- City/Province dictionary
- District dictionary per city/province

## Stage 2 — Web Crawling (Data Collection)

### Goal
Collect listing data and extract structured fields from HTML pages.

### Methods Used
- Iterating through city/district URL targets
- Parsing listing content with HTML parsing
- Regular expressions / string extraction where needed
- Building a unified tabular dataset


## Stage 3 — Data Preprocessing & Feature Engineering

### Key Problems Addressed
- Mixed-type and missing values (strings in numeric fields)
- Inconsistent price formats (HTML vs description vs price/m²)
- Special price values (e.g., `"Thỏa thuận"` → missing/None)
- Unit normalization (e.g., thousand / million / billion conversions)

### Typical Transformations
- Drop invalid/out-of-scope rows (e.g., non-house listings)
- Convert numeric fields (`area`, `rooms`, `floors`, etc.)
- Create clean price variables:
  - Split price tokens into `(value, unit)`
  - Convert into a consistent unit (commonly **billion VND**)
- Encode booleans (amenities/legal flags) into 0/1
- Export cleaned modeling dataset  
  (commonly saved as `Handled Data.xlsx`)


## Stage 4 — Modeling & Evaluation

### Goal
Train regression models to predict:
- **Target variable:** `Giá` (house price)

### Feature Selection
- Remove non-predictive / leakage / unstable fields where needed  
  (example: drop `Ngày đăng`, lat/long depending on experiment scope)

### Methods Used
#### A) PCA (Exploratory)
- PCA is used to analyze variance structure and dimensionality behavior of the feature set.
- Helps understand redundancy and complexity of the engineered features.

#### B) Regression Models Compared
The notebook evaluates several baseline and tree-based regressors, including:
- `LinearRegression`
- `Ridge`
- `Lasso`
- `DecisionTreeRegressor`
- `RandomForestRegressor`
- `GradientBoostingRegressor`

#### C) Train/Test Procedure
- `train_test_split` for hold-out evaluation

#### D) Metrics
Models are compared using:
- **MAE** (Mean Absolute Error)
- **MSE** (Mean Squared Error)
- **R²** (Coefficient of Determination)


## 6. Tech Stack
- **Python** (pandas, numpy, regex)
- **Web scraping** (BeautifulSoup / urllib)
- **Data preprocessing & feature engineering**
- **Machine learning** (scikit-learn regression models)
- **Evaluation & visualization** (matplotlib / seaborn)

---

## 7. Project Outcomes
- Built a fully automated pipeline to collect, clean, and structure real-world real estate data from the web at scale.
- Engineered high-quality features from noisy listing data and trained multiple regression models to accurately predict house prices.
- Delivered actionable pricing insights by identifying key property attributes driving price variation across locations.


