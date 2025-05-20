# Spatial-Temporal Person Re-identification 操作流程教學

本文件將詳細說明如何建立專案資料夾結構、下載資料集與預訓練模型、檔案放置位置，以及完整的執行流程與預期結果。

---

## 1. 資料夾結構

請依下列結構建立資料夾（部分資料夾需手動建立，部分會於執行程式時自動產生）：

```
Spatial-Temporal-Re-identification/
│
├── dataset/
│   ├── market_rename/
│   │   ├── gallery/
│   │   ├── query/
│   │   ├── train/
│   │   ├── train_all/
│   │   └── val/
│   ├── Market1501_prepare/
│   │   ├── gallery/
│   │   └── query/
│   └── DukeMTMC_prepare/
│       ├── gallery/
│       └── query/
│
├── model/
│   ├── ft_ResNet50_pcb_market_e/
│   │   ├── net_last.pth
│   │   └── pytorch_result2.mat
│   └── ft_ResNet50_pcb_duke_e/
│       ├── net_last.pth
│       └── pytorch_result2.mat
│
├── raw-dataset/
│   ├── Market-1501-v15.09.15/
│   └── DukeMTMC-reID/
│
├── ...（其餘原始碼與說明文件）
```

> **備註**：  
> - `dataset/` 內的 `market_rename/`、`Market1501_prepare/`、`DukeMTMC_prepare/` 會由 `prepare.py` 產生，需先放置原始資料集於 `raw-dataset/`。
> - `model/` 內的子資料夾與檔案可由訓練產生，或直接下載作者提供的預訓練模型。

---

## 2. 下載所需檔案

### (A) 資料集

1. **Market-1501**  
   - 官方下載連結：[Market-1501 Dataset](https://github.com/zhunzhong07/Market-1501)
   - 下載後解壓縮，放置於：  
     `raw-dataset/Market-1501-v15.09.15/`

2. **DukeMTMC-reID**  
   - 官方下載連結：[DukeMTMC-reID Dataset](https://github.com/layumi/DukeMTMC-reID_evaluation)
   - 下載後解壓縮，放置於：  
     `raw-dataset/DukeMTMC-reID/`

---

### (B) 預訓練模型（可選，若不想自行訓練）

- Google Drive 下載連結：  
  [Models(+RE)](https://drive.google.com/drive/folders/1FIreE0pUGiqLzppzz_f7gHw0kaXZb1kC)
- 下載後，將對應的 `.pth` 與 `.mat` 檔案放置於：  
  - `model/ft_ResNet50_pcb_market_e/`
  - `model/ft_ResNet50_pcb_duke_e/`

---

## 3. 資料集預處理

執行下列指令，將原始資料集轉換為專案所需格式：

- 處理 Market1501：
  ```sh
  python3 prepare.py --Market
  ```
- 處理 DukeMTMC-reID：
  ```sh
  python3 prepare.py --Duke
  ```

執行後，`dataset/` 內會產生對應的 `market_rename/`、`DukeMTMC_prepare/` 等資料夾。

---

## 4. 訓練模型（可跳過，若已下載預訓練模型）

- Market1501 訓練：
  ```sh
  python3 train_market.py --PCB --gpu_ids 0 --name ft_ResNet50_pcb_market_e --erasing_p 0.5 --train_all --data_dir "dataset/market_rename/"
  ```
- DukeMTMC-reID 訓練：
  ```sh
  python3 train_duke.py --PCB --gpu_ids 0 --name ft_ResNet50_pcb_duke_e --erasing_p 0.5 --train_all --data_dir "dataset/DukeMTMC_prepare/"
  ```

訓練完成後，模型權重會儲存於 `model/ft_ResNet50_pcb_market_e/` 或 `model/ft_ResNet50_pcb_duke_e/`。

---

## 5. 特徵提取與評估流程

以 Market1501 為例，完整流程如下：

1. **特徵提取**
   ```sh
   python3 test_st_market.py --PCB --gpu_ids 0 --name ft_ResNet50_pcb_market_e --test_dir "dataset/market_rename/"
   ```

2. **產生時空分布模型**
   ```sh
   python3 gen_st_model_market.py --name ft_ResNet50_pcb_market_e --data_dir "dataset/market_rename/"
   ```

3. **評估（joint metric）**
   ```sh
   python3 evaluate_st.py --name ft_ResNet50_pcb_market_e
   ```

4. **Re-ranking（可選）**
   ```sh
   python3 gen_rerank_all_scores_mat.py --name ft_ResNet50_pcb_market_e
   python3 evaluate_rerank_market.py --name ft_ResNet50_pcb_market_e
   ```

DukeMTMC-reID 流程同理，將 `market` 替換為 `duke`，指令參數與資料夾名稱相應更換。

---

## 6. 預期結果

- 執行 `evaluate_st.py` 後，終端機會顯示 Rank-1、mAP 等指標。
- 若執行 re-ranking，`evaluate_rerank_market.py` 會顯示 re-ranking 後的指標。
- 若使用預訓練模型，Market1501 可達 Rank@1 ≈ 98%、mAP ≈ 95%（re-ranking 後）。

---

## 7. 常見問題

- **GitHub push 失敗**：請確認 `.gitignore` 已忽略 `dataset/`、`model/`、`.pth`、`.mat` 等大型檔案。
- **資料夾結構錯誤**：請嚴格依照本文件建立資料夾與放置檔案。
- **模型下載失敗**：可嘗試 Google Drive 或百度雲備用連結。

---

如有其他細節，請參考 `note.md`、`complex_calc_note.md`、`evaluate_st_detail.md` 等補充說明文件。

---
