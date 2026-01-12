# HFT Price Movement Prediction (ETH/USDT)
https://www.notion.so/Overnight-Finance-Challenge-ETH-USDC-Predictions-25a2ab88fe6380f996e7c61fc9a9e036

[Russian Version](#russian-version) | [English Version](#english-version)

---

<a name="english-version"></a>
## English Version

### Project Overview
This repository contains a high-performing solution for an HFT (High-Frequency Trading) competition focused on predicting price movements of ETH/USDT based on Level 2 (L2) Limit Order Book (LOB) data. 

**Rank:** 17th on Leaderboard  https://docs.google.com/spreadsheets/d/1yquqp6Uhd3-8fVCGpQ4JRsFAjh8uGxOVnmpW8YS5G1E/edit?gid=0#gid=0

**Metric (F1-macro):** 0.458

### 1. Market Microstructure Analysis (EDA)
Before modeling, a deep dive into market dynamics was performed:
* **Impulse Response:** Calculated the cumulative return for each class. It confirmed that "Strong Up/Down" signals (classes 3 & 4) have significant predictive power over a 40-100 tick horizon.
* **Volatility Clustering:** Autocorrelation Function (ACF) analysis of absolute returns showed slow decay, justifying the use of multi-horizon rolling volatility features.
* **LOB Shape:** Visualized average liquidity distribution across 20 levels to calibrate the **Order Flow Imbalance (OFI)** decay rates.



### 2. Feature Engineering Pipeline (140+ Features)
The core of the solution lies in capturing microstructure alpha:
* **Multi-level Weighted OFI:** Computes the net flow of liquidity across 20 levels. 
    $$OFI_{total} = \sum_{i=1}^{20} w_i \cdot OFI_i, \text{ where } w_i = 0.9^i$$
* **Liquidity Center of Gravity (COG):** Measures the weighted average distance of liquidity from the best bid/ask. It identifies "hidden" pressure before it hits the mid-price.
* **Spoofing & Pressure:** * *Spoofing:* Ratio of L1 volume to L2 volume to detect fake walls.
    * *Energy:* Product of OFI and volatility to capture high-intensity flow.
* **Micro-Moments:** Liquidity absorption rate (burn rate), spread compression shocks, and "stuck" regime indicators (low movement periods).



### 3. Machine Learning Strategy
* **Model:** `CatBoostClassifier` (GPU-accelerated).
* **Class Imbalance:** Applied custom weights `[1, 3, 3, 8, 8]` to prioritize minority "Strong Move" classes.
* **Validation:** Time-Series split with a **200-tick gap (buffer)** to eliminate look-ahead bias caused by high autocorrelation in HFT data.
* **Threshold Optimization:** Instead of raw predictions, I performed a grid search for optimal probability thresholds ($T_{strong}$ and $T_{normal}$) to maximize the F1-macro score.

---

<a name="russian-version"></a>
## Russian Version

### Обзор проекта
В этом репозитории представлено решение задачи прогнозирования движения цены (ETH/USDT) на сверхкоротких временных интервалах на основе данных стакана (LOB) глубиной 20 уровней.
**Место:** 17 в лидерборде  
**Метрика (F1-macro):** 0.458

### 1. Анализ микроструктуры рынка (EDA)
Ключевые инсайты из анализа данных:
* **Impulse Response:** Построение средних траекторий цены подтвердило, что сигналы "Strong" (3 и 4 классы) обладают инерцией, которая сохраняется до 100 тиков.
* **Автокорреляция (ACF):** Анализ выявил слабую линейную зависимость доходностей, но очень сильную зависимость волатильности ("кластеризация"), что привело к созданию признаков волатильности на разных окнах.
* **Дисбаланс классов:** Нейтральный класс составляет >70% данных, что потребовало агрессивного взвешивания классов при обучении.

### 2. Feature Engineering (Более 140 признаков)
Основная ценность решения — в математическом описании динамики стакана:
* **Многоуровневый OFI (Order Flow Imbalance):** Агрегированный поток ордеров по 20 уровням с экспоненциальным затуханием весов.
* **Center of Gravity (COG):** Расчет "центра тяжести" ликвидности. Позволяет увидеть, на каком расстоянии от спреда сосредоточен основной объем.
* **Индикаторы поглощения и "сгорания":**
    * *Burn Rate:* Скорость исчезновения ликвидности на стороне Bid/Ask.
    * *Absorption:* Объем пассивной ликвидности, поглощающей рыночные ордера.
* **Сложные взаимодействия:** Энергия потока (OFI $\times$ Volatility), шоки спреда (отношение текущего спреда к MA) и относительная плотность уровней (L1 vs L5).

### 3. Стратегия моделирования
* **Алгоритм:** `CatBoost` на GPU. 
* **Взвешивание:** Для компенсации дисбаланса использованы кастомные веса `[1, 3, 3, 8, 8]`.
* **Валидация:** Валидация на временных рядах (последние 10%) с защитным буфером в 200 тиков, чтобы избежать утечки данных из-за автокорреляции.
* **Оптимизация порогов:** Ключевым этапом стал подбор оптимальных порогов вероятностей для классификации. Это позволило "выжать" дополнительные 0.01-0.02 к метрике F1 за счет более точного отделения сильных движений от рыночного шума.

---

### Как запустить / How to run
1. Скачать `train.csv` и `test.csv`.
2. Запустить ноутбук. Код включает автоматический поиск порогов (Threshold Search), который адаптируется под валидационную выборку.
3. Результат сохранится в `submission_final.csv`.
