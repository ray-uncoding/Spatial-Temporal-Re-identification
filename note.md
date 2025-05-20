# 專案程式碼架構與流程說明

本專案為「空間-時間人物再識別（Spatial-Temporal Person Re-identification, ST-ReID）」的完整實作，涵蓋資料準備、模型訓練、特徵提取、時空建模、評估與重排序等步驟。以下為各階段流程與主要檔案說明：

## 1. 資料準備（prepare.py）
- 依據不同資料集（Market1501 或 DukeMTMC-reID），將原始圖片依照訓練、驗證、查詢（query）、資料庫（gallery）分類，並依人物 ID 建立資料夾。
- Market1501 會進行額外的檔名重新命名，將時空資訊編碼進檔名，方便後續時空建模。

## 2. 模型訓練（train_market.py / train_duke.py）
- 使用 ResNet50、DenseNet121 或 PCB 等架構進行訓練。
- 支援資料增強（如 random erasing），提升模型泛化能力。
- 訓練完成後會將模型權重儲存於 model 目錄下。

## 3. 特徵提取與測試（test_st_market.py / test_st_duke.py）
- 載入訓練好的模型，對 query 與 gallery 圖片進行特徵提取。
- 會將每張圖片的特徵向量、標籤、攝影機編號、影格等資訊儲存為 .mat 檔案，供後續評估使用。

## 4. 時空分布建模（gen_st_model_market.py / gen_st_model_duke.py）
- 根據訓練集資料，統計不同攝影機間的時間差分布，建立時空分布模型（spatial-temporal distribution）。
- 輸出為 .mat 檔案，供評估時結合外觀特徵與時空資訊。

## 5. 評估（evaluate_st.py）
- 載入特徵與時空分布模型，計算查詢圖片與資料庫圖片的相似度（結合外觀特徵與時空分布）。
- 輸出 Rank-1、mAP 等指標，評估模型表現。

## 6. 重排序（Re-ranking）
- gen_rerank_all_scores_mat.py：計算所有查詢與資料庫圖片的分數矩陣，為 re-ranking 做準備。
- evaluate_rerank_market.py / evaluate_rerank_duke.py：利用 re-ranking 方法對檢索結果進行後處理，進一步提升檢索準確率。

## 7. 模型架構（model.py）
- 定義多種模型結構（ResNet50、DenseNet121、PCB），包含分類層設計與初始化方法，是訓練與測試時的核心網路架構。

---

## 流程總結
1. 準備資料（prepare.py）
2. 訓練模型（train_market.py / train_duke.py）
3. 特徵提取（test_st_market.py / test_st_duke.py）
4. 建立時空分布（gen_st_model_market.py / gen_st_model_duke.py）
5. 評估（evaluate_st.py）
6. 重排序與最終評估（gen_rerank_all_scores_mat.py、evaluate_rerank_market.py / evaluate_rerank_duke.py）

---

如需進一步了解每個步驟的細節，可參考對應的 .py 檔案註解或詢問具體流程！
