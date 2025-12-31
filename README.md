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

The first stage creates a systematic, scalable way to crawl across the entire country by generating URL patterns for **all provinces/cities and their districts**. Instead of manually hardcoding targets, the workflow builds a consistent mapping from URL “tails” (crawlable endpoints) to human-readable geographic names. This ensures that the crawler can iterate across the full geographic hierarchy and remain maintainable if the site structure changes slightly.

The notebook uses HTML parsing utilities (commonly **BeautifulSoup**) to discover and validate location endpoints. The key idea is to separate (a) URL construction logic and (b) meaning/labels that humans interpret. A **city/province dictionary** maps location identifiers to display names, and a **district dictionary per city/province** stores the nested structure needed to expand crawl coverage reliably.

This stage produces the dictionaries and URL target sets that are used as the “routing system” in the crawling stage. In practice, it becomes the backbone of an automated crawl plan: *for each city → for each district → crawl listing pages*.

---

## Stage 2 — Web Crawling (Data Collection)  

This stage collects listing data at scale and converts unstructured HTML pages into a structured dataset. The crawler iterates through the city/district targets created in Stage 1, fetches listing pages, and extracts standardized fields into a unified table.

The crawling process loops through all location targets, parses listing content using HTML parsing, and performs targeted string extraction (regular expressions or rule-based parsing) where the website does not provide clean structured tags. The project standardizes the extracted results into a single tabular dataset so downstream preprocessing and modeling can operate consistently.

The crawl exports an initial dataset (commonly stored as **Initial Data.xlsx** or a similar file). At this point the dataset is intentionally “raw but structured”: it contains important fields, but still includes missing values, mixed types, inconsistent price formats, and potentially out-of-scope listings that must be filtered later.

---

## Stage 3 — Data Preprocessing & Feature Engineering  
### Key Problems Addressed  
Real-world listing data is noisy. Numeric fields are often stored as text, units may vary, price may appear as “negotiable” (*Thỏa thuận*), and the same concept may be expressed differently across pages. This stage focuses on transforming raw collected fields into a consistent modeling dataset.

Mixed-type values are handled by converting numeric-like strings into numbers and treating non-parsable cases as missing. Price is the most complex field: it can appear in multiple formats (total price, price per m², or embedded in description) and in multiple units (thousand/million/billion VND). The preprocessing logic normalizes these representations into consistent variables so model training is meaningful and comparable across listings.

### Typical Transformations  
The pipeline removes invalid/out-of-scope rows (for example, non-house listings if the crawl endpoint includes multiple real-estate types). It cleans and converts area, room counts, floors, and other structural attributes into numeric columns. Price variables are reconstructed through tokenization and normalization: price text is split into components (value + unit), then converted into a single standard unit (commonly **billion VND**) for the regression target.

Binary signals—amenities and legal flags—are encoded into 0/1 features, ensuring they can be used by linear models and tree models without additional handling. Intermediate “data check” exports may be produced during iterative cleaning, and the final output is a cleaned modeling dataset commonly saved as **Handled Data.xlsx**.

A final modeling-ready dataset with:
- stable numeric features,
- encoded categorical/boolean indicators,
- a clean target variable representing house price in a consistent unit.

---

## Stage 4 — Modeling & Evaluation  

The final stage builds regression models to predict **house price** from engineered features, then evaluates performance using standard regression metrics.

### Feature Selection and Leakage Control  
Not all fields should be used as predictors. Some fields are non-predictive identifiers (listing ID, raw URL), some are unstable metadata (posting date), and some may introduce leakage or create an unrealistic evaluation setting depending on the experiment (e.g., latitude/longitude might be included or excluded depending on whether the goal is purely attribute-based pricing vs. location-aware pricing). The modeling notebook selects an appropriate feature set to ensure the results reflect realistic predictive performance.

### Methods Used  
The workflow includes **PCA as an exploratory step** to understand the variance structure and redundancy across engineered features. PCA is not necessarily used as the final model, but it helps explain dimensionality behavior and feature correlation patterns that influence model choice.

Multiple regressors are compared, including linear baselines and tree-based methods:
- LinearRegression
- Ridge
- Lasso
- DecisionTreeRegressor
- RandomForestRegressor
- GradientBoostingRegressor

A hold-out evaluation strategy is used via `train_test_split`, and performance is compared using:
- **MAE** (Mean Absolute Error)
- **MSE** (Mean Squared Error)
- **R²** (Coefficient of Determination)

A model comparison view that highlights which approach best captures pricing patterns given the engineered features, along with interpretable metrics that can guide further improvement (feature refinement, hyperparameter tuning, or richer location modeling).


## 6. Tech Stack
- **Python** (pandas, numpy, regex)
- **Web scraping** (BeautifulSoup / urllib)
- **Data preprocessing & feature engineering**
- **Machine learning** (scikit-learn regression models)
- **Evaluation & visualization** (matplotlib / seaborn)


## 7. Project Outcomes
- Built a fully automated pipeline to collect, clean, and structure real-world real estate data from the web at scale.
- Engineered high-quality features from noisy listing data and trained multiple regression models to accurately predict house prices.
- Delivered actionable pricing insights by identifying key property attributes driving price variation across locations.

##  Team Members
- Pham Le Hong Duc
- Truong Binh Ba
- Bach Ngoc Le Duy
- Hoang Minh Hien

