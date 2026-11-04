# W03 Lattice、Covert Channels、Non-Interference、Integrity 詳細解說

[← 回到目錄](./)

> 來源：`W03 Week_3_Slides.pdf`、study guide。**【小測】** = study guide 點名要識；「（補充）」= 我加的解釋。

## 1. Lattice：把 label 畫成圖【小測：要會畫】
- 上週講過：label 之間的 dominates 是**偏序**（partial order），因為有些 label 不能比較。
- 練習：(L,{})、(L,{A})、(L,{A,B})、(H,{A,B})、(H,{A,B,C}) 五個 label 能否排成一條直線（total order）？
  （補充）**可以**：(L,{}) ≤ (L,{A}) ≤ (L,{A,B}) ≤ (H,{A,B}) ≤ (H,{A,B,C})。
  加入 (L,{B,C}) 呢？它和 (L,{A})、(L,{A,B})、(H,{A,B}) 都不能比較 → **不能**再排成直線。
- 這些 label 在 dominates 之下形成一種叫 **lattice** 的結構，作用：(1) 按等級組織 label；(2) 控制資訊流向。

### 怎樣讀 lattice 圖
例子：等級 {H, L}（H > L），類別 {A, B}，共 8 個 label：

```
              (H,{A,B})
             ↗    ↑    ↖
       (H,{A})    │    (H,{B})
          ↑     (H,{})   ↑
          │   (L,{A,B})  │
       (L,{A})     ↑    (L,{B})
             ↖    │    ↗
               (L,{})
```
（簡化示意。原圖的 (H,{}) 指向 (H,{A}) 和 (H,{B})；(L,{A,B}) 指向 (H,{A,B})；(L,{}) 也指向 (H,{})。）

- **箭頭由低指向高**。從 ℓ1 有路徑走到 ℓ2，代表 ℓ2 ≥ ℓ1，也代表**資訊可以由 ℓ1 流到 ℓ2**。
- 有些箭頭沒畫，原因有兩個：(1) 兩個 label 不能比較；(2) **遞移性**（transitivity）：A ≤ B 且 B ≤ C，就推得 A ≤ C，不用再畫 A→C。
- 資訊由 ℓ1 流到 ℓ2 的兩種方式：(1) ℓ2 的 subject 讀 ℓ1 的 object；(2) ℓ1 的 subject 寫入 ℓ2 的 object。沒有路徑時，simple security property 擋 (1)，*-property 擋 (2)。
- **BLP 的目標：資訊在 lattice 中只可以向上流**（只有 ℓ2 ≥ ℓ1 時才可以由 ℓ1 流到 ℓ2）。其他方向都是違規。

### 練習：填入缺少的 label
圖中已有：頂 (TS,{Air,Sea})、(TS,{Air})、(Confidential,{Sea})、底 (Unclassified,{})。要填入 A (Confidential,{Air})、B (TS,{Sea})、C (Secret,{Air,Sea})。

（補充）答案和推理：
- 左下（由 Unclassified 指向，再指向 (TS,{Air}) 和中間）→ **A (Confidential,{Air})**
- 中間（由兩個 Confidential 指向，再指向頂）→ **C (Secret,{Air,Sea})**：它同時 dominate (C,{Air}) 和 (C,{Sea})
- 右上（由 (C,{Sea}) 指向，再指向頂）→ **B (TS,{Sea})**
- 技巧：類別越多、等級越高，就越在上面。先找「誰 dominate 誰」，再按順序擺。

### BLP 夠不夠？
問題：一個系統能否**完全遵守 BLP 的規則，但資訊仍然向下流**（違反 metapolicy）？答案是可以 → 這就是 covert channel。這也說明 metapolicy 可以用來檢驗規則夠不夠。

## 2. Covert channel（隱蔽通道）【小測】
### 經典例子：CREATE／DESTROY
系統除了 READ／WRITE，還有：
- CREATE(S, O)：如果 O 不存在，就在 S 的等級建立 O；否則甚麼都不做。
- DESTROY(S, O)：如果 O 存在而且 S 可以寫它，就刪除。
這兩個操作看起來都符合 BLP。

**高等級 S_H 怎樣把 1 bit 傳給低等級 S_L：**
- 想傳 0 → S_H 執行 Create(S_H, F0)，在高等級建立 F0
- 想傳 1 → S_H 甚麼都不做

（補充，slides 只寫了結果）S_L 之後依次做：Create(S_L, F0) → Write(S_L, F0, 1) → Read(S_L, F0)。
- 如果 F0 已由 S_H 建立（高等級）：S_L 的 create 失敗，讀時又不可以 read up → 回傳 **0**
- 如果 F0 不存在：S_L 在低等級建立 F0，寫入 1，讀回 **1**

所以 S_L 看到的值，就是 S_H 想傳的 bit。重複做，就能傳任意多的資料。

**為甚麼 BLP 擋不住**：BLP 只把 object 的**內容**當成資訊的載體，但這裡的資訊藏在「S_L 能不能讀到一個叫 F0 的 object」這個答案裡。

### 定義
> **Covert channel**：系統內 subject 之間的非法資訊流，利用了本來不是設計來給 subject 互相溝通的系統資源。

- 它違反 **metapolicy**，但**不一定違反 policy**（BLP 規則照樣遵守）。
- 資訊流發生在系統內的 subject 之間。
- 只要 S_L 會因為 S_H 的不同行為而看到不同結果，就可以用來傳 bit。

### 兩大類：storage vs timing【小測：要會分類】

| 類型 | 資訊藏在哪 | 例子 |
|---|---|---|
| **Covert storage channel** | **系統狀態**（某個屬性的值） | 例 1：S_L 嘗試存取高等級資源，錯誤訊息是「找不到資源」還是「拒絕存取」，由 S_H 控制，每次傳 1 bit |
| **Covert timing channel** | **事件的次序或時間長短** | 例 2：KVM/370 輪流分配 CPU 時間，S_H 用盡自己的時間代表 1，立即讓出代表 0；S_L 看時鐘知道過了多久 |

- 例 3（磁碟）：S_H 和 S_L 共用磁碟，磁頭先服務最近的磁柱。S_H 讀第 140 或 160 磁柱，S_L 請求 139 和 161，S_L 收到結果的**先後次序**就透露了 S_H 讀過哪裡。（補充）這是靠事件次序，通常歸為 timing channel。
- 例 4（implicit channel，隱含通道）：`if h == 1 then l = 1`，l 的值透露了 h 的值，靠的是程式的控制流程。
- 其他類型：timing、implicit、termination（會不會結束）、probability、resource exhaustion（資源會否耗盡）、power（耗電量）。實務上通常只分 storage 和 timing。
- 判斷技巧（補充）：問「接收方看的是**一個值**，還是**時間或次序**？」看值 → storage；看時間或次序 → timing。

### 特性和處理方法
- 速度不慢：真實處理器上的 covert channel 每秒可傳**數千 bit**，對系統表現沒有明顯影響。
- 三個特性：**existence**（是否存在）、**bandwidth**（每秒傳多少）、**noisy／noiseless**（有沒有失真）。
- 真實系統幾乎不可能消除所有 covert channel。發現一個後可以：
  1. **Eliminate**：改系統實作，關掉它
  2. **Reduce bandwidth**：加入雜訊
  3. **Monitor**：監察使用模式，例如入侵偵測
- **Storage channel 的成立條件**：雙方都能接觸某個共享 object 的屬性；發送方能**修改**它；接收方能**看到**它；有機制啟動雙方並安排先後次序。
- **Timing channel 的條件**：雙方都能接觸共享屬性；雙方都有時間參考（時鐘、計時器、事件次序）；發送方能控制接收方察覺變化的時間；有啟動和排序的機制。
- 現代例子：**Meltdown**（利用亂序執行，在權限檢查前讀到 kernel 記憶體）、**Foreshadow**（利用亂序執行讀 L1 cache）。

## 3. 偵測：Shared Resource Matrix（SRM）【小測：要會解釋，不用自己做】
Kemmerer 在 1980 年代初提出。根據系統的功能規格，畫一張表：**行 = 共享屬性，列 = 操作**。
- **R**：該操作在某些情況下會**透露**（reference）這個屬性的資訊
- **M**：該操作在某些情況下會**修改**（modify）這個屬性

|  | READ | WRITE | DESTROY | CREATE |
|---|---|---|---|---|
| File existence | R | | M | M |
| File size | R | M | M | M |
| File level | R | | M | M |

- **同一行同時有 R 和 M → 可能有 covert channel**：有操作可以改它（發送方），也有操作可以看它（接收方）。
- 例如 CREATE：做完之後你就知道檔案存在了，所以 CREATE 對 file existence 也算 R。重點是「操作有沒有透露屬性的資訊」。
- SRM 只**指出要去哪裡找**，不會直接證實 covert channel；每張表只適用於一個系統。
- 實際做法：用 BLP 管一般的資訊流 → 用 SRM 找 covert channel → 關掉、限制或監察它們。限制：要非常熟悉系統操作的語意和實作。

## 4. Non-Interference（NI）【小測】
- 想法：如果 S_H 絕對不能和 S_L 溝通，那 S_H 做的**任何事**都不應該對 S_L 產生可見的影響。
- NI 是 **information flow policy**（資訊流政策）的代表：不規定 subject 可以讀寫哪些 object，而是規定**哪個 subject 可以 interfere（影響）哪個 subject**。
  - interfere = 做了某件事，對方看得到效果。
- 政策寫成 subject 之間的關係，例如 S1 → S2、S2 → S3。

### NI 和 MLS（BLP）的關係
- **任何 MLS 政策都可以轉成 NI 政策**：S_i → S_j 當且僅當 S_j 的等級 dominate S_i。
  例子：A (Secret:{Crypto,Nuclear})、B (Secret:{Crypto})、C (Secret:{})（slides 印成 {Unclassified}）→ C→B、C→A、B→A。
- **反過來不成立**：NI 政策可以**不遞移**（例如 S1→S2、S2→S3，但沒有 S1→S3），而 MLS 一定遞移。
- 例子：防火牆。Internet → Firewall、Firewall → LAN，但**不要** Internet 直接 → LAN。

### NI 處理了 BLP 處理不到的東西
- NI 直接表達 metapolicy（「S_H 不能影響 S_L」），沒有關於讀寫 object 的規則。
- 要證明系統符合 NI，就要證明 S_H 的任何行為都不會改變 S_L 看到的東西。S_L 的「視野」包括越多，政策越強：
  - 只包括 BLP 下 S_L 看得到的檔案內容 → 等於 BLP
  - 加入系統旗標 → 擋住 storage channel
  - 加入系統時鐘 → 擋住 timing channel
  - 包括 S_L 能觀察到的一切 → 沒有任何 covert channel
- 為甚麼不把一切都包括？真實系統的 interference 太多，大多在底層，很多是無害的（例如加密檔案），所以**要證明真實系統符合 NI 極度困難**。

## 5. Integrity 入門
- **Confidentiality**：誰可以讀？**Integrity**：誰可以寫、修改或產生資訊？**Availability**：需要時用得到嗎？
- Integrity 比 confidentiality 更模糊、更視乎情況。
- **和機密不同**：程式即使不和外界互動，只要計錯數，就已經破壞了 integrity。
- 例子：「網紅將誕下外星人的嬰兒」這個標題你信不信？要看來源是《紐約時報》還是八卦小報。你判斷的是**來源的可信程度**。

### Integrity label
- **Object 的 label**：資訊有多可信（八卦 < 專家報告）。
- **Subject 的 label**：我們多信任它能正確產生或處理資訊（經認證的程式 > 網上下載的免費軟件）。
- 形式和 BLP 一樣：等級 + 類別。例如物理教授 (Expert: {Physics})：在物理上可信，但不代表他對政治的意見也可信。
- **Integrity label 和 clearance 是兩回事**：同時要機密和完整性的系統，每個 subject 和 object 要有兩套 label。資料可以「很敏感但不太可靠」，也可以「很可靠但不敏感」。
- Dominates 一樣適用：(Expert:{Physics}) ≥ (Student:{Physics}) 是；(Novice:{Physics,Arts}) vs (Expert:{Physics}) 否；(Student:{Art}) ≥ (Novice:{}) 是。

### 商業世界的 integrity 原則
- **Separation of duty**：關鍵工作要由幾個不同的人一起完成。
- **Separation of function**：同一個人不能擔任同一流程中互補的角色。
- **Auditing**：保留審計紀錄，方便還原和追究責任。
- 商業的控制多數是 discretionary、按程序、分散的，不像軍方那樣強制和集中；而且**商業上 integrity 往往比 confidentiality 更重要**。
- **Lipner 的 4 個商業 integrity 需求**：
  1. 用戶不自己寫程式，只用現成的正式軟件。
  2. 程式員在非正式系統上開發和測試，可以用假資料。
  3. 把程式由開發環境搬到正式環境要經特別程序，並受控制和審計。
  4. 經理和審計員要能看到系統狀態和系統紀錄。

### Integrity 的 metapolicy
- **不要讓壞資訊「污染」好資訊**，即**資訊不能在 integrity 上向上流**。
- 壞資訊流入好 object 的兩種方式：(1) 低 integrity 的 subject **寫入**高 integrity 的 object；(2) 高 integrity 的 subject **讀入**低 integrity 的 object。
- 所以規則和 BLP 相反：**no write up、no read down**（W04 的 Biba 模型）。
- 機密和完整性是**正交**（orthogonal）的問題，要分開處理。

## 6. 自我測驗
1. 兩個程序共用印表機佇列。高等級程序放不放文件入佇列，低等級程序看佇列長度得知。這是 storage 還是 timing channel？
2. 高等級程序故意令 CPU 很忙，低等級程序量度自己的運算慢了多少。哪一類？
3. SRM 表中，某一行只有 R 沒有 M，代表甚麼？
4. 為甚麼 NI 能處理 covert channel 而 BLP 不能？

**答案**：
1. **Storage**：接收方看的是一個值（佇列長度）。
2. **Timing**：接收方看的是時間。
3. 沒有操作可以修改那個屬性，發送方無法用它傳訊息 → 不會是 covert channel。
4. BLP 只管讀寫 object 的內容；NI 規定 S_H 的**任何行為**都不能影響 S_L 看到的東西，包括系統屬性和時間。

---

[← W02 Policy 和 BLP](w02-policy-blp.md) ｜ [目錄](./) ｜ [W04 Integrity 模型、RBAC →](w04-integrity-models-rbac.md)
