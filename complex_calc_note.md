# evaluate_st.py 計算流程說明

本檔案負責將外觀特徵與時空分布結合，計算查詢圖片與資料庫圖片的最終相似度分數，並評估 Rank-1、mAP 等指標。

## 主要計算流程

### 1. 特徵正規化
- 將 query 與 gallery 的特徵向量做 L2 normalization，確保特徵向量長度一致。

### 2. 外觀特徵分數計算
- 使用 `np.dot(gf, query)` 計算查詢特徵與所有 gallery 特徵的餘弦相似度。

### 3. 時空分布分數計算
- 針對每一個 gallery 圖片，根據查詢與 gallery 的攝影機編號、影格時間差，查表取得時空分布機率（distribution）。
- 以 100 為單位將時間差分箱（histogram binning），查詢對應的分布值。

### 4. 分數融合
- 將外觀分數與時空分布分數以 sigmoid 函數壓縮後相乘：
  
  `score = 1/(1+exp(-alpha*score_appearance)) * 1/(1+2*exp(-alpha*score_st))`
- 這樣可同時考慮外觀與時空資訊。

### 5. 排序與評估
- 根據最終分數降序排序，計算 CMC（累積匹配特徵曲線）與 mAP（mean Average Precision）。
- good/junk index 的設計可排除同攝影機或無效樣本。

### 6. 時空分布平滑
- 對時空分布進行高斯平滑（gauss_smooth2），避免分布過於稀疏。

---

# gen_st_model_market.py 計算流程說明

本檔案負責統計訓練集不同攝影機間的時間差分布，建立時空分布模型。

## 主要計算流程

### 1. 解析檔名取得 ID、攝影機、影格
- 依據重新命名後的檔名，解析出每張圖的 ID、攝影機編號、影格時間。

### 2. 統計每個 ID 在每個攝影機的平均時間
- 對每個 ID、每個攝影機，累加影格時間並計算平均值。

### 3. 建立時間差分布
- 對每個 ID，兩兩攝影機組合，計算平均時間差，並以 100 為單位分箱，累加到對應的分布矩陣。
- 分布矩陣 shape 為 (8,8,max_hist)，代表 8 個攝影機間的時間差分布。

### 4. 分布正規化
- 對每個攝影機組合的分布做歸一化，確保為機率分布。

---

# gen_rerank_all_scores_mat.py 計算流程說明

本檔案負責計算所有查詢與資料庫圖片的分數矩陣，為 re-ranking 做準備。

## 主要計算流程

### 1. 特徵與時空分布正規化
- 與 evaluate_st.py 相同，先做特徵正規化與時空分布平滑。

### 2. 分數計算
- 對所有圖片（query+gallery）兩兩計算融合分數（外觀+時空），存成 all_scores 矩陣。

### 3. 輸出
- 將 all_scores 儲存為 .mat 檔案，供 re-ranking 使用。

---

# evaluate_rerank_market.py 計算流程說明

本檔案負責利用 re-ranking 方法對檢索結果進行後處理，提升檢索準確率。

## 主要計算流程

### 1. 載入 all_scores
- 讀取所有查詢與資料庫圖片的分數矩陣。

### 2. re-ranking 計算
- 調用 re_ranking.py 的 re_ranking 函數，根據分數矩陣進行 re-ranking，考慮 k-reciprocal 最近鄰等資訊。

### 3. 評估
- 以 re-ranking 後的分數進行排序與評估，計算 Rank-1、mAP 等指標。

---

# 補充說明

這些流程的核心目的是：
- 先利用外觀特徵與時空分布計算所有圖片間的相似分數（all_scores），
- 再透過 re-ranking（如 k-reciprocal 最近鄰）進行排序優化，
- 最終提升人物再識別的檢索準確率。

如需更細節的公式推導或流程圖，可再進一步詢問！
