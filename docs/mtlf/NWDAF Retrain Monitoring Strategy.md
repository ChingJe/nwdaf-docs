# NWDAF Retrain / Monitoring Strategy

## 前情提要：[Drift Monitor](https://www.notion.so/Drift-Monitor-3489318f11a1804a94a0e1bf68f43a3d?pvs=21)

---

## 為什麼需要修改

- 目前策略過度依賴單一 metric + 固定 threshold
- sMAPE 在低流量 / near-zero 時不穩
- threshold 很難定，代表問題可能不只在 metric，而在 decision policy 本身

## 目前策略的問題

- metric 同時負責
    - 描述誤差
    - 判定 drift
    - 觸發 retrain
- 問題在於這三件事其實不是同一層
    - 誤差變大，不一定代表資料型態真的變了
    - 資料型態真的變了，也不一定代表當下就值得立刻 retrain

## 論文中的設計思路

- 論文在 drift monitor 這邊，流程上大概是
    - 先把最近一段時間的 losses 存下來
    - 再根據這些 recent losses 算一些統計量
        - 例如平均值、標準差 / 變異
    - 接著拿當前想判斷的資料的 loss
        - 去和前面 observed 到的結果做比較
    - 看目前 loss 是否已經偏離過去的常態很多
- 所以它本質上做的事情不是單看這次 error 大不大
    - 而是看這次 error 相對於 recent baseline，是否已經異常偏移
- 換句話說
    - 它先建立一個「最近正常表現大概長怎樣」的基準
        - 再拿目前 loss 去檢查是不是明顯偏離這個基準

---

## 目前比較在乎的幾個設計點

### metric 選型

- metric 先不要定死
    - 不需要一開始就決定最後一定是 MAE / MSE / WAPE / NRMSE 哪一個
    - 比較合理的做法是先把它們都當成候選
        - MAE
        - MSE
        - WAPE
        - NRMSE
    - 可以在同一批 monitor data 上進行測試
        - 先觀察它們各自的穩定性、分布、可解釋性
            - recent history 長什麼樣
            - 是否容易受低流量影響
            - 數值是否容易解讀
            - threshold / z-score 是否好設
            - …
        - 再決定最後要拿哪一個當主 metric
    - 這樣比較不會太早被某個指標綁住

### path-level 與 model-level 的關係

- monitor 雖然是作用在 model 身上，但 metric 不應該先在 model 內全部混成單一值（目前做法）
    - 如果同一個 model 服務多個 group / 區域 / path
        - 那不同 path 的 prediction quality 可能差很多
    - 若先把所有 path 的資料混在一起算一個 aggregate metric
        - 很容易讓某一條真的退化的 path 被其他正常 path 稀釋掉
    - 因此比較合理的方式是
        - **先做 path-level metric calculation**
        - 再回到 model-level 做 decision
    - 也就是說
        - monitor 掛在 model 上
        - 但 metric / drift calculation 要先在 path 上分開算
- model-level retrain decision 可以建立在 path-level 結果上
    - 這裡比較符合需求的方向是
        - 只要其中一條 path 明顯出問題，就讓這個 model retrain
    - 所以 decision 邏輯比較像
        - 各 path 分開監控，任一 path 達到條件
            - model-level retrain trigger 就成立
    - 這樣比先做全體平均（目前NWDAF的實作）更符合實際需求

### drift statistic 的形式

- drift monitor 的 statistic 先採用比較可解釋的形式
    - 一開始不一定要照論文的公式原封不動搬
    - 先用容易理解、也比較容易 debug 的形式會比較好
    - 例如：
        - recent buffer 的平均值
        - recent buffer 的標準差
        - 看 current error 高了 recent mean 幾個標準差 (z-score)
    - 這種寫法的好處是
        - 可解釋
        - 容易畫圖
        - 也比較容易調參
- drift monitor 應該建立在 recent baseline 上
    - 核心概念不是單看這次 error 大不大
        - 而是看這次 error 相對於最近觀察到的常態，有沒有明顯偏離
    - 所以需要保留 recent metric buffer
        - 讓每條 path 都有自己的 recent history
    - 再由這個 history 算出 baseline
        - mean
        - std
        - 或其他簡單 summary statistics

### retrain decision 的基本原則

- 不能只因為「相對離群」就 retrain
    - 即使某次 error 相對於 baseline 偏了很多也不代表一定需要 retrain
    - 因為有可能是
        - baseline 本來就很低
        - std 也很小
        - 導致 current error 雖然絕對值不大，但 z-score 看起來很高
    - 這種情況下，如果 prediction 整體表現其實還很好
        - 不應該因為它成為離群值就觸發 retrain
- 因此 retrain decision 應該至少分成兩層
    - 第一層先看
        - **error 本身是否已高到值得關心**
    - 第二層再看
        - 它是否相對於 recent baseline 明顯偏離
    - 也就是說
        - **metric 低時，就算它相對上是離群值，也不應該 retrain**
        - 必須是
            - error 本身夠高且相對 baseline 偏離夠多才值得進 retrain decision
- 第一層進入判斷的門檻也可以考慮做成動態的
    - 不一定要是一個全域固定值
        - 可以隨 recent baseline 的 mean / std 去調整
    - 這樣比較能對應不同 path、不同 traffic pattern 的差異
        - 也比較符合這整套 recent-baseline-based monitor 的方向
- 整體上，目前比較傾向的方向是
    - metric 作為候選集合，先觀察再決定
    - path-level 分開計算 metric
        - 每條 path 也要建自己的 recent baseline
    - drift statistic 先用可解釋的 mean / std 形式 (z-score)
    - retrain 不只看相對離群，也要看 error 本身是否高到值得處理