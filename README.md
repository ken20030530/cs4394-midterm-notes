# CS4394 Information Security and Management：Midterm（W1–W6）溫習解說

CityU CS4394（2026/27 Semester A）第 10 週短小測（midterm）範圍是 **W1–W6**。這裡逐週用中文詳細解說 lecture slides，對照課程的 study guide，標出要識的重點，並附自我測驗。

> **非官方筆記**：由 Claude（AI）根據課程的 lecture slides、tutorial、in-class exercise 和 study guide 整理，再經人手使用。可能有錯，以課程原本的資料為準。這個 repo 不包含任何課程原檔（slides、PDF）。

## 目錄

| 週 | 主題 | 解說 |
|---|---|---|
| W01 | Security 基本詞、risk management、ALE、Ford Pinto、CIA 等 5 個原則 | [W01 Introduction](W01%20Introduction.md) |
| W02 | Metapolicy vs policy、security label、dominates、BLP、tranquility、MAC vs DAC、ACM | [W02 Metapolicy, Policy and BLP](W02%20Metapolicy%2C%20Policy%20and%20BLP.md) |
| W03 | Lattice、covert channel（storage／timing）、SRM、non-interference、integrity 入門 | [W03 Lattice, Covert Channels, Non-Interference, Integrity](W03%20Lattice%2C%20Covert%20Channels%2C%20Non-Interference%2C%20Integrity.md) |
| W04 | Biba 3 個政策、Lipner、Clark-Wilson、Chinese Wall、RBAC、ACL／capability、CISS | [W04 Integrity Models, Chinese Wall, RBAC](W04%20Integrity%20Models%2C%20Chinese%20Wall%2C%20RBAC.md) |
| W05 | ORCON 和 DRM、TCB 和 reference monitor、5 種攻擊、one-time pad、對稱 vs 非對稱、hash、MAC、AE | [W05 ORCON, TCB, Cryptography Basics](W05%20ORCON%2C%20TCB%2C%20Cryptography%20Basics.md) |
| W06 | RSA、digital signature、Google Cloud 的 3 種加密狀態和金鑰層級（DEK／KEK）、certificate、chain of trust | [W06 Public Key, Google Cloud Encryption, Certificates](W06%20Public%20Key%2C%20Google%20Cloud%20Encryption%2C%20Certificates.md) |

## 怎樣用
1. 每份解說內標 **【小測】** 的，是 study guide 點名要識的內容，先溫這些。
2. 標「（補充）」的是額外解釋，slides 沒有寫。
3. 每份最後有自我測驗和答案，溫完先自己答，再對答案。

## 最常考的計算和判斷（快速索引）
- **ALE = SLE × ARO**，措施價值 = ALE 前 − ALE 後 − 每年成本 → W01
- **Dominates**：等級 ≥ **而且** 類別 ⊇ → W02
- **BLP**：讀要「我 ≥ 它」（no read up）；寫要「它 ≥ 我」（no write down）→ W02
- **Biba Strict**：讀要「它 ≥ 我」（no read down）；寫要「我 ≥ 它」（no write up）→ W04
- **畫 lattice、填缺少的 label** → W03
- **Covert channel 分 storage 還是 timing**：看接收方看的是值，還是時間或次序 → W03
- **Chinese Wall 讀寫判斷** → W04
- **5 種密碼攻擊分類** → W05
- **金鑰數量**：對稱 n(n−1)/2；非對稱 2n → W05
- **DEK／KEK 外洩怎樣處理** → W06
