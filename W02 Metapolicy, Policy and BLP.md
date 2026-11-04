# W02 Metapolicy、Policy 和 Bell-LaPadula 詳細解說

> 來源：`W02 Week_2_Slides.pdf`、`W02 Week_2_answers_to_examples.pdf`、Tutorial 1 Q4–Q8、study guide。
> **【小測】** = study guide 點名要識；「（補充）」= 我加的解釋。

## 1. Security policy 是甚麼
- 不同系統對「安全」的要求不同（手機網絡、軍事資料庫、網上銀行），所以要為每個系統寫清楚規則。
- **Policy = 一套為了達成特定安全目標的規則**。可以把它想成系統設計者和客戶之間的合約：規則要能做到（achievable），也要足夠達成目標（adequate）。

### 例子：Anne 和 Bill 抄功課
規則是「不准抄功課」。Anne 忘記把功課檔案設為不可讀，Bill 抄了。誰違反了 security？
- **Bill 違反了**：規則禁止抄，他抄了，系統進入了「未授權狀態」（Bill 有一份 Anne 的功課）。
- **Anne 沒有違反**：規則沒有要求她保護檔案。如果規則寫了「學生必須把功課設為不可讀」，Anne 也算違反。
- 重點：**有沒有違反 security，是對照 policy 判斷的**，不是對照常識。

## 2. Metapolicy vs Policy【小測】
大學有 3 條規則：職員不可在文件中用學生的 HKID；有 HKID 的文件除非有需要否則要銷毀；必須保留的要鎖好。這些規則背後的「大目標」是甚麼？保護學生個人資料不被洩露。這個大目標就是 metapolicy。

| | **Metapolicy** | **Policy** |
|---|---|---|
| 層次 | 高層次，整體的安全目標 | 針對某個系統的具體規則 |
| 回答 | **What 和 why**（要保護甚麼、為甚麼） | **How**（怎樣做到） |
| 用詞 | 通常用 CIA 來描述 | 用具體機制：防火牆、加密、上鎖的抽屜 |
| 比喻 | 框架 | 規則手冊 |

- 沒有清楚的 metapolicy，policy 看起來就像隨意定的，也無法評估規則對不對。
- 既然 metapolicy 這麼重要，為甚麼還要 policy？因為 metapolicy 太籠統、可以有多種解讀、可以有多套 policy 都達到同一目標；policy 才能給用戶和開發者具體、可執行的指引。
- **BLP 的例子（Tutorial 1 Q6）**：metapolicy = 資訊只可以向上流（保護機密）；policy = no read up、no write down 兩條規則。

## 3. Multi-Level Security（MLS）：給資料貼標籤
背景：1943 年艾森豪將軍的辦公室。資料有不同敏感程度：壘球比賽時間、飯堂今天吃漢堡、某上校加薪、諾曼第登陸日期、英國破解了 Enigma。顯然不是所有資料同樣敏感，所以要分級。

### Security label = 等級 + 類別【小測】
- **等級（hierarchical level）**：有大小次序。Unclassified < Confidential < Secret < Top Secret。
- **需知類別（need-to-know categories）**：沒有次序的集合，例如 {Crypto, Nuclear, Personnel}。
- 寫法：**(Secret: {Nuclear, Crypto})**。
- **文件（object）的 label** 表示它有多敏感；**人（subject）的 label 叫 clearance**，表示他獲准看哪類資料。兩者形式相同。
- 一份文件混合了不同內容：等級用**最高**那個，類別**全部包括**。
- 改變文件的等級叫 downgrading 或 upgrading。

### 為甚麼要有類別：Principle of Least Privilege（PoLP）【小測】
**任何 subject 只應該得到完成工作所需的最少資料。**即使同是 Top Secret，也不是人人都需要知道所有事。這幾乎是 security 的公理。

## 4. Dominates 關係【小測：要會比較】
術語：**Object** = 被保護的資料容器（文件、檔案、資料庫）；**Subject** = 發出請求的用戶或程序；**Action** = 對 object 做的操作。

> **(L1, S1) dominates (L2, S2)** 當且僅當：
> 1. L1 ≥ L2（等級不低於對方），**而且**
> 2. S1 ⊇ S2（類別是對方的超集，即包含對方所有類別）
> 寫作 (L1, S1) ≥ (L2, S2)。

**兩個條件都要成立。** 這是 partial order（偏序）：有些 label 互相都不 dominate 對方，叫 **incomparable（不可比較）**。

例子（slides＋答案）：

| Label 1 | Label 2 | 1 dominates 2？ | 原因 |
|---|---|---|---|
| (Secret: {Crypto}) | (Confidential: {Crypto}) | **是** | Secret ≥ Confidential，{Crypto} ⊇ {Crypto} |
| (Secret: {Crypto, Nuclear}) | (Top Secret: {Crypto}) | **否** | Secret < Top Secret，等級不夠 |
| (Secret: {Nuclear}) | (Unclassified: {}) | **是** | 等級夠，任何集合都 ⊇ 空集 |

練習（L2 > L1），哪個 dominate？（slides 沒有附答案，以下按定義推出）
- (L1, {NUC}) vs (L2, {NUC, CRYP}) → **(L2, {NUC, CRYP})**
- (L2, {RECON}) vs (L1, {RECON}) → **(L2, {RECON})**
- (L2, {CRYP}) vs (L2, {NUC, CRYP, RECON}) → **(L2, {NUC, CRYP, RECON})**
- (L1, {NUC, CRYP}) vs (L1, {NUC}) → **(L1, {NUC, CRYP})**

**Incomparable 例子（Tutorial 1 Q4）**：(H, {A, B}) 和 (H, {C, D})：等級一樣，但兩邊都不包含對方的類別 → 不可比較。

## 5. BLP 的兩條規則【小測：要會判斷 R／W／RW／N/A】

### Simple Security Property = no read up（只可以讀下）
> Subject 可以**讀** object，**只有當 subject 的 clearance dominates object 的 label**。

例子：Alice 的 clearance 是 (Secret: {Crypto})。
- A (Confidential: {Crypto}) → 可讀
- B (Secret: {Crypto}) → 可讀
- C (Secret: {Nuclear}) → 不可讀：她沒有 Nuclear 類別
- D (Top Secret: {Crypto}) → 不可讀：等級不夠

### *-Property（star property）= no write down（只可以寫上）
> Subject 可以**寫** object，**只有當 object 的 label dominates subject 的 clearance**（方向和讀相反）。

為甚麼要限制「寫」？（補充）防止有人（或有木馬的程式）把高機密資料抄進低機密文件，讓低等級的人看到。Slides 強調：電腦上的 subject 可能是一個程式，用戶通過了審查，但程式沒有，所以寫入一定要管。

例子（slides＋答案）：

| Subject | Object | 可寫？ | 原因 |
|---|---|---|---|
| (Secret: {Crypto}) | (Confidential: {Crypto}) | 否 | 向下寫 |
| (Secret: {Crypto, Nuclear}) | (Top Secret: {Crypto}) | **否** | 等級是向上寫，可以；但 object 的 {Crypto} 不包含 subject 的 Nuclear，可能把 Nuclear 資料寫進只屬 Crypto 的文件 |
| (Secret: {Nuclear}) | (Unclassified: {}) | 否 | 向下寫 |

**記法**：讀 → 我要 ≥ 它；寫 → 它要 ≥ 我。

### BLP 的副作用（slides 指出）
- 將軍（Top Secret）不能用電郵把命令發給沒有 clearance 的士兵（向下寫）。
- 但沒有 clearance 的士兵**可以**覆寫作戰計劃（向上寫是允許的）！這是 **integrity** 問題，BLP 只管機密，不管完整性 → W04 的 Biba 模型處理這個。

### 要會填 Access Control Matrix（ACM）【小測】
三個 subject：Subj1 (H, {A,B,C})、Subj2 (L, {})、Subj3 (L, {A,B,C})；
三個 object：Obj1 (L, {A,B,C})、Obj2 (L, {})、Obj3 (L, {B,C})；H > L。

|  | Obj1 (L,{A,B,C}) | Obj2 (L,{}) | Obj3 (L,{B,C}) |
|---|---|---|---|
| **Subj1** (H,{A,B,C}) | R | R | R |
| **Subj2** (L,{}) | W | RW | W |
| **Subj3** (L,{A,B,C}) | RW | R | R |

做法（補充）：每一格問兩條問題：
1. Subject ≥ object？→ 可讀（R）
2. Object ≥ subject？→ 可寫（W）
兩樣都是 → RW（只有兩個 label 完全相同時才會出現）；兩樣都不是 → N/A。
例如 Subj2 對 Obj1：{} 不包含 {A,B,C} → 不可讀；(L,{A,B,C}) ≥ (L,{}) → 可寫 → **W**。

## 6. Tranquility：label 可以改嗎？【小測】
如果可以隨意把 (Top Secret: {Crypto}) 降到 (Unclassified: {Crypto})，就等於繞過了 no write down。所以要多一條規則：
- **Strong tranquility**：系統運行期間，subject 和 object 的 label **永遠不改**。
- **Weak tranquility**：label 可以改，但**不能違反 policy 的精神**。

Tutorial 1 Q5 的答案：
- **提高** object 的等級（假設當時沒人在讀寫）：strong tranquility 下違反（因為 label 不准改）；從 no read up／no write down 的角度看不違反（資訊沒有往下流）。但如果有個 Secret 用戶正在讀，就提升到 Top Secret，就變成 read up。
- 把 Top Secret 分區的資料**完全銷毀後**降為 Unclassified：strong tranquility 下不准；weak tranquility 下可以，因為已沒有敏感資料會洩露。

**BLP 模型 = Simple Security Property + *-Property + Tranquility**。由 Bell 和 LaPadula 在 1973–75 年為美國國防部設計，專注**機密性**，至今仍是電腦安全的基石。

## 7. Access control policy：MAC vs DAC【小測】

| | **MAC（Mandatory）** | **DAC（Discretionary）** |
|---|---|---|
| 規則 | 每次存取都強制檢查，**用戶不能改** | 某些用戶（通常是擁有者）可以放寬或修改 |
| 例子 | **BLP**：不符合兩條規則的存取一律拒絕 | Unix 檔案權限（擁有者可以 chmod）、Windows NTFS 權限 |

- 任何 access control policy 都可以畫成 ACM：行是 subject，列是 object，格內是允許的動作。
- 但真實系統太大，矩陣會非常巨大。BLP 的矩陣「隱含」在兩條規則內，需要時才即場計算。

## 8. 自我測驗
1. (TS: {A}) 和 (S: {A, B}) 誰 dominate 誰？
2. Subject (S: {A, B})，object (S: {A})：R、W、RW 還是 N/A？
3. 為甚麼 BLP 擋不住「低等級的人破壞高等級文件」？
4. 「所有員工要把電腦鎖屏」是 policy 還是 metapolicy？

**答案**：
1. 都不 dominate：TS 等級較高，但 {A} 不包含 B → incomparable。
2. Subject ≥ object（S ≥ S，{A,B} ⊇ {A}）→ 可讀；object ≥ subject？{A} 不包含 B → 不可寫 → **R**。
3. *-property 允許向上寫，BLP 只保護機密不保護完整性。
4. Policy（具體做法）。背後的 metapolicy 可能是「防止未授權的人接觸公司資料」。
