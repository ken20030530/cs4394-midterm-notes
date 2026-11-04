# W04 Integrity 模型（Biba、Lipner、Clark-Wilson）、Chinese Wall、RBAC 詳細解說

[← 回到目錄](./)

> 來源：`W04 Week_4_Slides.pdf`、Tutorial 2（答案）、Exercises 3（答案，10/29 新版）、study guide。
> **【小測】** = study guide 點名要識；「（補充）」= 我加的解釋。

## 1. Biba 的 3 個 integrity 政策（1977）【小測：每個的讀寫規則】
3 個政策都替 subject 和 object 貼 **integrity label**（類似 BLP 的 clearance），差別在於**對 subject 有多信任**。

### (a) Strict Integrity（一般說的「Biba model」）= BLP 的 dual（對偶）
- **Simple Integrity Property**：subject 只可以**讀** integrity **≥ 自己**的 object → **no read down**。理由：讀了壞資料，自己會被污染。
- **Integrity *-Property**：subject 只可以**寫** integrity **≤ 自己**的 object → **no write up**。理由：不能用自己的資料污染更可靠的資料。
- 為甚麼叫 dual（Tutorial 2 Q1）：label 的用法和 BLP 一樣，但**規則方向剛好相反**。

| | 讀 | 寫 |
|---|---|---|
| **BLP**（機密） | no read **up**（我 ≥ 它） | no write **down**（它 ≥ 我） |
| **Biba Strict**（完整） | no read **down**（它 ≥ 我） | no write **up**（我 ≥ 它） |

**ACM 練習（slides 只給題目）**：H > L。Subj1 (H,{A,B,C})、Subj2 (L,{})、Subj3 (L,{A,B})；Obj1 (L,{A,B,C})、Obj2 (L,{})、Obj3 (L,{B,C})。
（補充）按 Biba Strict 推出：

|  | Obj1 (L,{A,B,C}) | Obj2 (L,{}) | Obj3 (L,{B,C}) |
|---|---|---|---|
| **Subj1** (H,{A,B,C}) | W | W | W |
| **Subj2** (L,{}) | R | RW | R |
| **Subj3** (L,{A,B}) | R | W | N/A |

每格問：object ≥ subject？→ R；subject ≥ object？→ W。例如 Subj3 對 Obj3：{B,C} 不包含 A，{A,B} 也不包含 C → 互不 dominate → N/A。

### BLP + Biba 一起用【小測】
同時要機密和完整 → 每個 subject 和 object 要有**兩套 label**；一個存取要 **BLP 和 Biba 都允許**才可以。
（補充）做題步驟：先用 confidentiality label 按 BLP 判斷 R／W，再用 integrity label 按 Biba 判斷 R／W，**兩邊都有的才保留**。Study guide 寫「見 Exercises 5 Q6」，但新版 Exercises 5 是加密題，這類題目在 Tutorial 2 Q5（Lipner）。

### (b) Low Water Mark（LWM）
- 讀：**可以讀任何 object**，但讀完後 subject 的 integrity **降到 min(自己, object)**。
- 寫：只可以寫 integrity ≤ 自己的 object（no write up）。
- 假設：subject 會被讀到的東西「污染」，所以讀了低 integrity 的資料就降級。
- 問題：**label creep**。Subject 越讀越低，最後沒有 subject 能存取高 integrity 的 object，分析變得過度保守。
- Exercises 3 Q3：高 integrity 的分析員讀了低 integrity 的新聞來源 → 他的 integrity 降到新聞來源的等級；目的是防止他之後把受污染的結論寫回高 integrity 的 object。

### (c) Ring Policy
- 讀：**任何 subject 可以讀任何 object**，不論 integrity。
- 寫：只可以寫 integrity ≤ 自己的 object。
- 假設：subject **能自己過濾**收到的資料，所以最信任 subject。

### 3 個政策的比較（Tutorial 2 Q3、Q4；Exercises 3 Q4）

| 高 integrity 的 subject 讀低 integrity 的 object | 結果 | 對 subject 的信任 |
|---|---|---|
| Ring | 可以讀，沒有後果 | **最高** |
| LWM | 可以讀，但 subject 降級 | 中 |
| Strict | **不准讀** | **最低** |

- **共通點**：3 個都**禁止 write up**。
- Tutorial 2 Q2：除了 Chinese Wall，**LWM** 也會按過往存取紀錄調整等級。

## 2. Lipner 的 Integrity Matrix Model【小測：一般描述】
- 處理 W03 講的 4 個商業需求：用戶不自己寫程式；程式員在非正式系統開發；開發搬到正式要經受控和審計的程序；經理和審計員要看系統狀態和紀錄。
- 做法：**BLP + Biba Strict 混合**，每個 subject／object 有兩套 label。
- **Confidentiality**：等級 AM（Audit Manager）> SL（System Low）；類別 SP（production）、SD（development）、SSD（system development）。
- **Integrity**：等級 ISP（system program）> IO（operational）> ISL（system low）；類別 ID（development）、IP（production）。

| Subject | Confidentiality | Integrity |
|---|---|---|
| 一般用戶 | (SL, {SP}) | (ISL, {IP}) |
| 應用程式開發者 | (SL, {SD}) | (ISL, {ID}) |
| 系統程式員 | (SL, {SSD}) | (ISL, {ID}) |
| 系統經理／審計員 | (AM, {SP, SD, SSD}) | (ISL, {IP, ID}) |
| 系統控制員 | (SL, {SP, SD})＋**downgrade** | (ISP, {IP, ID}) |

| Object | Confidentiality | Integrity |
|---|---|---|
| 開發中的程式／測試資料 | (SL, {SD}) | (ISL, {ID}) |
| 正式程式 | (SL, {SP}) | (IO, {IP}) |
| 正式資料 | (SL, {SP}) | (ISL, {IP}) |
| 軟件工具 | (SL, {}) | (IO, {ID}) |
| 系統程式 | (SL, {}) | (ISP, {IP, ID}) |
| 修改中的系統程式 | (SL, {SSD}) | (ISL, {ID}) |
| 系統和應用程式紀錄 | (AM, {SP, SD, SSD}) | (ISL, {}) |

- 一般用戶要修改**正式資料**（同級，可以），但不能修改**正式程式**（integrity IO 比用戶高，Biba 禁止 write up）。為了這點，confidentiality 那邊拿走了寫入限制。
- Slides 的問題和答案：
  - 一般用戶可以用系統程式嗎？可以修改嗎？→ 「用」即讀，**可以讀，不能修改**。
  - 系統程式員可以用或修改正式程式嗎？→ **兩樣都不可以**（類別 SSD 和 SP 互不包含）。
  - 為甚麼要特別的 **downgrade** 權限？→ 把程式由開發搬到正式，就要**改 label**，BLP 和 Biba 本身做不到（違反 tranquility），所以要特別授權給系統控制員。
- **Tutorial 2 Q5**：一般用戶 (a) 讀正式程式：機密同級可讀，integrity 用戶較低所以可以讀上 → **可以**；(b) 寫系統紀錄：紀錄的機密 label dominate 用戶 → 可以寫上，integrity 用戶 ≥ 紀錄 → 可以寫 → **可以**。
- 結論：可行，但不直觀 → 也許需要全新的模型 → Clark-Wilson。

## 3. Clark-Wilson 模型（1987）【小測：重點】
- 想法：商業安全有自己的需要，最重要是系統狀態的**一致性**。Integrity 由一組**限制條件**定義，資料符合這些條件就是 consistent／valid。
- 銀行例子：日初資金 X + 存入 Y − 提取 Z = 日終資金 W。**Well-formed transaction** 必須令系統由一個一致的狀態去到另一個一致的狀態。誰去檢查和認證交易做得對？

### 4 個基本關注【小測】
1. **Authentication**：所有用戶的身份要正確驗證。
2. **Audit**：每次修改都要記錄執行了甚麼程式、由誰執行，而且紀錄不能被破壞。
3. **Well-formed transactions**：用戶只能以受限的方式處理資料。
4. **Separation of duty**：每個用戶只可以執行指定的程式，防止未授權的修改。

### 4 類元素【小測】

| 元素 | 意思 | 銀行例子 |
|---|---|---|
| **CDI**（Constrained Data Item） | 受 integrity 保護的資料 | 戶口結餘 |
| **UDI**（Unconstrained Data Item） | 不受保護的資料 | 用戶在鍵盤輸入的數字 |
| **TP**（Transformation Procedure） | **唯一**可以修改 CDI，或把用戶輸入變成新 CDI 的程序；令系統由一個有效狀態去到另一個 | 存款、轉帳程序 |
| **IVP**（Integrity Verification Procedure） | 檢查 CDI 是否仍然有效 | 日終對帳 |

### Certification vs Enforcement 規則【小測：分別和目的】
- **Certification rules（C）**：由人（認證者）檢查並認證，**應用系統**在交易時要遵守的要求。
- **Enforcement rules（E）**：由**系統的保護機制**自動執行的要求。（Tutorial 2 Q6a）

| 規則 | 內容 |
|---|---|
| C1 | 執行 IVP 時，必須確保所有 CDI 處於有效狀態 |
| C2 | 每個 TP 要被認證為能保持 integrity：把一組有效的 CDI 轉成另一個有效狀態（定義了 TP 和 CDI 的 **certified relation**） |
| C3 | 分配 TP 給用戶（**allowed relation**）要符合 separation of duty |
| **C4** | **TP 的運作必須記錄**：把足夠重建該操作的資料**附加到一個只可附加（append-only）的 CDI**，即 log，讓審計員知道發生了甚麼 |
| C5 | 以 UDI 為輸入的 TP，對任何可能的輸入值，只能做有效的轉換或不轉換：要麼拒絕，要麼轉成 CDI。例如鍵盤輸入的數字要先驗證 |
| E1 | 只有獲認證的 TP 可以處理 CDI（系統維護 certified relation） |
| E2 | 用戶只能經自己獲授權的 TP 存取 CDI（系統維護 allowed relation） |
| E3 | 每個嘗試執行 TP 的用戶都要經**身份驗證**（用系統不一定要驗證，但處理 CDI 之前一定要） |
| **E4** | **只有 TP 的認證者可以修改與該 TP 相關的實體名單**；而認證者**永遠不可以執行**他認證過的東西 → 落實 separation of duty |

- **E4 的重要性**（Exercises 3 Q6；Tutorial 2 Q6b）：認證 TP 的人和執行 TP 的人必須是不同的人 → **防止欺詐**：沒有一個人能既批准又執行一宗惡意交易而不被發現。
- **C4 的重要性**（Exercises 3 Q8）：TP 更新戶口結餘時，要把足以重建操作的資料附加到 append-only 的 log。
- **注意**：Exercises 3 Q7（新版問 E3）的官方答案說 E3「強制用 triple，令用戶不能直接改 CDI」，內容其實較接近 E2。（補充）建議答題時兩點都寫：E3 要求執行 TP 前驗證身份，再配合 triple 確保用戶只能經授權的 TP 改 CDI。

### Access triple【小測】
- 允許的權限寫成 **(user, TP, {CDI set})**：這個用戶可以用這個 TP 處理這組 CDI。
- 每個 triple 都要符合所有適用的 C 和 E 規則。

### UDI 怎樣變成可信（Exercises 3 Q2、Q9）
外來、不可信的資料（例如電郵收到的 CSV）先當 **UDI**，經**獲認證的 TP**（驗證程序）轉換後才成為 **CDI**。

### 和 Biba 比較
- **Biba**：沒有 certification 規則；靠「受信任的 subject」確保行為守規矩；不可信的資料要經檢查才給高 label。
- **Clark-Wilson**：明確寫出行為要符合的要求；受信任的人認證的是**升級資料的方法**（TP），**不是資料本身**。

## 4. Chinese Wall 政策（Brewer & Nash, 1989）【小測】
- 處理**利益衝突**：顧問或承包商不小心洩露資料。例子：律師替國泰做顧問，再替香港航空做就有衝突，因為她給其中一間的意見會影響另一間；但同時替麥當勞做沒有問題。
- 嚴格來說它**不是 integrity 政策**，而是 **access control 的 confidentiality 政策**。

### 3 層抽象【小測】
1. **Object**：一個檔案只關於一間公司。
2. **Company dataset（CD）**：某公司的所有 object。
3. **Conflict of interest class（COI）**：互相競爭的公司的 dataset 組成一個 COI。例如 {Ford, Chrysler, GM}、{HSBC, 渣打, Citibank}、{Microsoft}。

### 讀：CW Simple Security Property【小測：要會用】
Subject 可以讀 object，只要該 object 符合以下任何一項：
- 和 subject **已經讀過**的 object 在**同一個 CD**（在牆內）；**或**
- 屬於一個**完全不同的 COI**（subject 從沒碰過的類別）；**或**
- 是**已清理（sanitized）**的公開資料。

例子：讀過 GM 的檔案，之後就**永遠不能**讀 Ford 或 Chrysler，但可以讀任何銀行或 Microsoft。
- 權限是**動態**的，取決於**過往存取紀錄**（history）。一開始可以讀任何東西。
- **怎樣處理利益衝突**：一旦讀了某 COI 內一間公司的資料，同一 COI 的其他公司就被擋住，所以不會同時知道兩間競爭公司的機密。

### 寫：CW *-Property【小測】
Subject 可以寫 object，**當且僅當**：
1. CW simple security 允許它讀這個 object；**而且**
2. 它能讀的**所有未清理 object 都屬於同一個 CD**。

為甚麼要第 2 條：Anthony 可以讀 Bank 1 和 Gas 的 CD，Susan 可以讀 Bank 2 和 Gas 的 CD。如果 Anthony 可以寫入 Gas 的 CD，他就可以把 Bank 1 的資料抄進去，Susan 再讀 → 間接知道 Bank 1 的資料，造成利益衝突。
Exercises 3 Q5：要寫入某 CD，subject 必須 (1) 能讀那個 CD，而且 (2) 沒有讀過同一 COI 內其他公司的資料。

### 和 BLP、Clark-Wilson 比較
- CW **沒有 security label**，BLP 有；CW 有**過往存取紀錄**，BLP 沒有。
- BLP 可以表示某一刻的狀態（每對 (COI, CD) 當一個類別），但**無法追蹤時間上的變化**。例如 Susan 病了要 Anna 接手，CW 的紀錄能判斷 Anna 可不可以接手，BLP 做不到。
- 和 Clark-Wilson：如果一個人可以用多個程序（process），他可以用不同程序繞過 CW；所以 subject 要定義成「一個人和他執行的所有程序」。

## 5. Role-Based Access Control（RBAC）【小測】
- 權限綁定**角色**（工作職能），不是綁定個人。角色例子：銀行的總裁、經理、培訓員、櫃員、審計員、清潔工。
- 每個人有：**authorized roles**（獲准擔任的角色）和 **active roles**（現在擔任的角色）。每個角色有一組 **transactions**（可做的業務操作，例如開戶、兌現支票、轉帳）。

### 3 條主要規則【小測】
1. **Role assignment**：subject 要有 active role 才可以執行 transaction。
2. **Role authorization**：subject 的 active role 必須是他的 authorized role。
3. **Transaction authorization**：subject 只可以執行他某個 active role 獲准的 transaction。

- 一個人可以有多個角色，例如經理有需要時可以當櫃員。
- **Subsumption**：一個角色可以包含另一個，例如培訓員能做受訓者的所有事，還有更多。
- **Separation of duty**【小測】：同一個人不能同時擔任某兩個角色。例如他的 authorized roles 有櫃員，就不能有審計員。目的：防止一個人既做事又審核自己，即防止欺詐和錯誤。
- **優點**：容易管理（同一角色的人權限相同）；權限用機構的語言（「開戶」而不是「讀檔」）；一個人可以轉換角色而不用換身份。
- **Role engineering**：定義角色和所需權限，常用於兩間機構合併時。**Role mining**：分析現有權限，找出最佳的角色分配；這是 NP-complete，但實務上可以近似。

## 6. 3 種儲存 ACM 的方法【小測】
明確地儲存整個 ACM 太昂貴：矩陣通常很稀疏，大部分格是空的（Exercises 3 Q10a）。

| 方法 | 存在哪 | 說明 |
|---|---|---|
| **規則即場計算** | 規則 | 按 subject 和 object 的屬性即場計算，例如 BLP；Exercises 3 答案叫它 attribute-based access control |
| **Access Control List（ACL）** | 跟 **object** | 每個 object 一張表：<subject, 權限> |
| **Capability** | 跟 **subject** | 每個 subject 持有 <object, 權限>，像門票，出示就能存取 |

- Capability 的風險：持有就等於有權限，不再另外檢查，所以**必須防止偽造或修改**。歷史上的做法：每個記憶體位置多一個 bit 標明是不是 capability（只有 OS 能處理），或放在受保護的記憶體。

## 7. Clinical Information Systems Security Policy（CISS）
- 用於病歷。重點是**病人私隱、病歷和寫病歷的人的身份驗證、integrity**，利益衝突不是重點。
- 角色：**病人**；**個人健康資料**（能辨認病人的健康或治療資料）；**臨床人員**。

| 原則 | 內容 |
|---|---|
| Access 1 | 每份病歷有 ACL，列出可以讀和**附加**資料的人；只有名單上的人可以存取。審計員只拿副本，不能改 |
| Access 2 | 名單上要有一位臨床人員有權加人入名單：**responsible clinician** |
| Access 3 | 打開病歷時，responsible clinician 要通知病人名單上有誰；除法律要求或緊急情況外，要取得病人同意 |
| Access 4 | 記錄每次存取的臨床人員、日期和時間；刪除也要記錄。資料只更新、不刪除，方便審計 |
| Creation | 臨床人員開病歷時，自己和病人都在名單上；因轉介而開的，轉介的醫生也可以在名單上 |
| Deletion | 病歷要過了適當時間才可刪除 |
| **Confinement** | 一份病歷的資料可以附加到另一份，**只有當第二份的 ACL 是第一份 ACL 的子集**，防止資料洩露給未授權的人 |
| **Aggregation** | 要有效防止聚合病人資料；有人要被加入名單（尤其是能存取大量病歷的人）時要通知病人。怕有人收集大量病歷、交叉比對，用來勒索 |
| Enforcement | 處理病歷的系統必須有子系統執行上述原則，並由獨立審計員評估 |

- 和 BLP 比較：confinement 原則令資料形成類似 lattice 的結構；CISS 著重**被存取的 object**，BLP 著重 subject。
- 和 Clark-Wilson 比較：CDI = 病歷；TP = 更新病歷和 ACL 的功能；IVP 核對臨床人員的身份、資料經驗證、要通知的人已被通知、要同意的已取得同意；C4 的審計 → 病歷只可附加，名單改變時通知病人。

## 8. 自我測驗
1. Subject (Mid, {X})，object (High, {X})，用 Biba Strict：可讀？可寫？
2. 同上，改用 Ring policy？
3. 律師讀過 HSBC 的資料，現在想讀渣打和 Microsoft 的資料，Chinese Wall 怎樣判？
4. 銀行規定「開支票的人不能是批核支票的人」，是 RBAC 的哪個概念？對應 Clark-Wilson 哪條規則？
5. 一間公司有 10,000 個用戶、100 萬個檔案，為甚麼不直接存 ACM？

**答案**：
1. Object 的 integrity 較高 → **可讀**（不是 read down）；**不可寫**（會 write up）。
2. Ring：任何讀都可以 → 可讀；寫仍然 no write up → 不可寫。
3. 渣打和 HSBC 在同一 COI → **不可以**；Microsoft 在另一個 COI → **可以**。
4. Separation of duty；Clark-Wilson 的 **C3**（分配 TP 要符合 separation of duty）和 **E4**（認證者不能執行）。
5. 矩陣有 100 億格而且大部分是空的，太浪費；改用 ACL、capability 或即場計算的規則。

---

[← W03 Lattice、Covert Channels](w03-lattice-covert-channels.md) ｜ [目錄](./) ｜ [W05 ORCON、TCB、密碼學基礎 →](w05-orcon-tcb-crypto.md)
