# W06 Public Key 加密、Google Cloud 加密、Certificate 和 Chain of Trust 詳細解說

[← 回到目錄](./)

> 來源：`W06 Week_6_Slides.pdf`、Exercises 5（答案，第 8 週做但考 W06 內容）、study guide。
> **【小測】** = study guide 點名要識；「（補充）」= 我加的解釋。

## 1. Public key（非對稱）加密【小測：RSA 靠甚麼、解決甚麼問題】
- 歷史：1976 年 **Diffie 和 Hellman** 公開提出加密和解密用不同金鑰。1997 年才披露英國 GCHQ 在 1970 年代初已發明，叫「non-secret encryption」。
- 用公開的 public key 加密，用秘密的 private key 解密：M = { {M}K }K⁻¹。
- 基礎是一個 **one-way function（單向函數）**：**容易計算，但沒有額外資料就很難倒推**。

### RSA【小測】
- Rivest、Shamir、Adleman 在 1970 年代末發明，靠**質因數分解（prime factorization）很難**。
- 把兩個大質數 p、q 相乘得 N = p × q 很容易；但只知道 N，要找回 p 和 q **非常困難**。如果已知 N 和其中一個質數，除一除就得到另一個。
- **為甚麼安全**（Exercises 5 Q5）：大質數相乘很容易，但由乘積還原質數在數字很大時計算上不可行。N 很小時可以逐個除來找；N 有幾百位數時實際上做不到。
- （補充）p、q 就是那個「額外資料」：持有 private key 的人等於知道它們，所以能倒推；其他人不知道，就做不到。

### 效率和用途
- Public key 運算可以比對稱加密**慢約 10,000 倍**，因為用的是複雜的數學運算，不是簡單的 bit 運算。
- 所以**對稱加密仍然是主力**，public key 負責幾件特別的事：
  1. **Key distribution（派發金鑰）**
  2. **Digital signature**：提供 authenticity 和 non-repudiation
  3. **由交換的 public key 衍生共同秘密**：**Diffie-Hellman** 協定，讓雙方經公開通道建立共同的秘密
  4. 實務上一定要有 **PKI**：可信的機構建立和管理 digital certificate，用來證明某條 public key 屬於某個實體。
- **Public key 解決的問題**【小測】：**key distribution problem**。不用預先經秘密通道共享金鑰，任何人都可以用你公開的 public key 加密給你。

## 2. Digital signature【小測：是甚麼、用途】
- 簽署：用**簽署者的 private key** 為訊息 M 產生簽名 sig = Sign(SK, M)。
- 驗證：任何人用**簽署者的 public key** 驗證：Verify(PK, M, sig) = 1；訊息被改成 M′（M′ ≠ M）就 = 0。
- **用途**：
  - **Authenticity（真確性）**：證明訊息真的由持有 private key 的人發出。
  - **Integrity**：訊息被改過就驗證失敗。
  - **Non-repudiation（不可否認）**：只有簽署者有 private key，所以他事後不能否認曾簽過。
- **Exercises 5 Q1**：Alice 用自己的 private key 簽 M，Bob 用 **Alice 的 public key** 驗證。M 簽署後被改過，驗證會**失敗**，因為簽名綁定原文的 hash，改了訊息 hash 就完全不同。
- （補充）加密和簽署方向相反，很易混淆：
  - **加密給 Bob**：用 **Bob 的 public key** 加密，Bob 用自己的 private key 解密 → 保護機密
  - **Alice 簽署**：用 **Alice 的 private key** 簽，任何人用 Alice 的 public key 驗證 → 證明來源

### 應用例子
- 家庭和商業：手機、藍光機、收費電視解碼器、電錶、互聯網（TLS、PGP、SSH）、磁碟加密、門卡、車匙、防盜系統。
- 軍事：敵我識別、戰術無線電、抗干擾雷達、武器解鎖碼、核彈頭的 permissive action link、導航訊號。
- 銀行：卡驗證碼、PIN 驗證、轉帳、網上銀行、電子錢包、數碼貨幣、加密資產。

## 3. Google Cloud 的加密【小測：重點題】
### 資料的 3 種狀態【小測：要會分類和舉例】

| 狀態 | 保護甚麼 | 例子 |
|---|---|---|
| **At rest（靜止）** | 資料**儲存**時，防止系統被入侵或資料被偷運出去 | 硬碟上的檔案用 **AES** 加密 |
| **In transit（傳輸中）** | 資料在兩方之間**傳送**時，防止被截取 | **TLS**（網絡傳輸）、**S/MIME**（電郵） |
| **In use（使用中）** | 伺服器**運算**資料時 | **Homomorphic encryption**（同態加密，可以直接對加密資料運算） |

Exercises 5 Q2：(a) 硬碟上的檔案被加密 → at rest；(b) 用 TLS 保護客戶端和伺服器之間的資料 → in transit；(c) 伺服器運算時保護資料 → in use。

### Encryption at rest 的好處
- 多一層 **defense in depth**：攻擊者沒有金鑰就讀不到資料。
- **縮小攻擊面**：等於「切走」底層的硬件和軟件；即使底層被攻破（例如有人實際拿到硬碟），加密得當的資料仍然安全。
- 透過 key management 控制和**審計**資料存取。
- 保障客戶私隱：系統可以做備份、工程師可以維護基建，但**看不到內容**。
- **Customer content**（客戶產生或提供的資料，例如 Cloud Storage 的資料、Compute Engine 的磁碟快照、IAM 政策）一律在 at rest 時加密；**customer metadata**（例如專案號碼、時間戳、IP 位址、物件大小、機器類型）只做到合理程度的保護。

### Storage system 層的加密
- 資料切成 **chunk**（每塊最多幾 GB），**每個 chunk 用自己的金鑰**加密，兩個 chunk 不會共用金鑰。
- **為甚麼要切 chunk、每塊一條 DEK**（Exercises 5 Q3）：**data isolation**，縮小金鑰外洩的「**blast radius**（波及範圍）」：一條金鑰外洩只影響那一塊。
- Chunk 更新時用**新金鑰**重新加密，不重用舊金鑰。
- 資料**寫入磁碟之前**就加密：加密是儲存系統本身的一部分，不是事後才加上。
- 每個 chunk 有唯一 ID，用 **ACL** 確保只有在那個時間獲授權的 Google 服務能解密。
- Chunk 分散儲存，並以加密形式複製作備份和災難復原。攻擊者要拿到資料，**既要所有相關的 chunk，也要每個 chunk 的金鑰**。
- 預設用 **AES-256**（NIST 建議長期儲存使用）。Mode：大部分用 **AES-GCM**；部分情況用 AES-CBC + HMAC；部分複製檔案用 AES-CTR + HMAC。

### Storage device 層和備份
- 大部分資料在**硬碟／SSD 層**再加密一次：用 AES-256（少數舊 HDD 用 AES-128），用**另一條**device 層的金鑰，和 storage 層的金鑰不同。
- 備份過程中資料一直保持加密；每個備份檔用自己的 DEK 加密（由 KMS 的一條金鑰加上備份時隨機產生的 seed 衍生）；備份的 metadata 另用一條 DEK。

## 4. 金鑰種類和金鑰層級【小測：哪條鑰匙加密哪條】
### DEK 和 KEK
- **DEK（Data Encryption Key）**：用來加密 **data chunk**；**存放在資料旁邊**，但本身被 **KEK 加密（wrap）**。
- **KEK（Key Encryption Key）**：用來**加密 DEK**；每個 Google Cloud 服務有一條或多條 KEK，數量比 DEK **少很多**；集中存放在 **KMS（Key Management Service）**。
- 好處：大規模儲存和加密資料時容易管理，也能追蹤和控制存取。

### 為甚麼要分層（Exercises 5 Q9，三個答案都對）
1. **限制金鑰外洩的影響**：DEK 外洩只影響它加密的資料，其他用不同 DEK 的資料不受影響。
2. **方便換金鑰**：換 KEK 時，只要用新 KEK **重新包裝 DEK**，不用把所有資料解密再加密，大大減少運算。
3. **可擴展**：資料量極大時，每組資料一條 DEK，再用少量 KEK 保護。

### 產生金鑰
- 全部用 Google 的共用密碼學程式庫產生；PRNG 的 seed 來自 Linux kernel 的 RNG，而它的 seed 來自多個獨立的熵來源：機房環境的隨機事件（例如磁碟尋道時間、封包到達間隔）和 Intel 的 RDRAND 指令。
- **KEK 設計上不能匯出 KMS**：所有用 KEK 的加密和解密都要在 KMS 內做，防止外洩和濫用，KMS 也可以記錄每次使用。

### Key rotation（換金鑰）
- KMS 可以定期自動換 KEK。資料用一個 **key set** 保護：**一條現役金鑰負責加密**，加上**一組舊金鑰負責解密**。
- Cloud Storage 每 **90 天**換一次 KEK，最多保留 **20 個版本**，所以資料最少每 **5 年**要重新加密一次（90 天 × 20 ≈ 5 年）。
- KMS 的金鑰有備份作災難復原。
- **Exercises 5 Q6**：定期換金鑰可以限制每條金鑰的使用時間，減少金鑰外洩的影響。但**換金鑰不能保證之前已外洩的資料安全**：攻擊者已經用舊金鑰取得明文的話，換金鑰也挽回不了。

### KEK 的存取控制
- 每條 KEK 在 KMS 有自己的 **ACL** 和政策，只有獲授權的 Google 服務和用戶可以用。
- **每次使用都要驗證身份並記錄**，所有資料存取都可以審計。

### 讀取一個加密 chunk 的流程
1. 服務向儲存系統要資料；儲存系統找出資料所在的 chunk ID。
2. 對每個 chunk，儲存系統取出和它存在一起的**已包裝 DEK**，送去 KMS 解包。
3. 儲存系統根據 job ID 和 chunk ID，核對這個 job 是否獲准存取這個 chunk。
4. KMS 核對儲存系統是否獲准使用該服務的 KEK，以及解包這條 DEK。
5. KMS 把**解包後的 DEK** 交回儲存系統，儲存系統解密 chunk 再交給服務。

### 整個金鑰層級（root of trust）【小測】
```
data chunk        ← 被 DEK 加密（DEK 和 chunk 存在一起）
  DEK             ← 被 KEK 包裝（KEK 在 KMS）
    KEK           ← 被 KMS master key 包裝（AES-256，存在 Root KMS）
      KMS master key ← 被 root KMS master key 包裝（AES-256）
        root KMS master key ← 存在 root KMS master key distributor（只在 RAM）
```
- **Root KMS**：只存大約十多條金鑰；不在一般正式機器上運行，只在每個機房的**專用機器**上運行。
- **Root KMS master key distributor**：**peer-to-peer** 架構，和 Root KMS 一對一地在同一批專用機器上運行，金鑰**只存在 RAM**，並用紀錄核對使用是否正確。新的 distributor 啟動時，從已在運行的其他 distributor 取得金鑰。
- **Gossiping protocol**：每個 distributor 定時隨機揀另一個比對金鑰版本，修正差異。**沒有中央節點**是整個系統都依賴的 → 高可用、低延遲。
- KMS 本身在全球機房複製了**數千次**。
- **災難復原**：root KMS master key 另外備份在安全的硬件裝置，放在兩個相隔很遠的 Google 地點的**實體保險箱**。只有在所有 distributor 同時停機時才需要（例如全球重啟）；能打開保險箱的 Google 員工**少於 20 人**。

### 金鑰外洩怎樣處理【小測：in-class exercise 有教】
- **DEK 外洩**（Exercises 5 Q10）：換上新 DEK 後，要**用新 DEK 重新加密受影響的資料**，並**停用舊 DEK**。只換金鑰不會把已用舊金鑰加密的資料重新加密。
- **KEK 外洩**（Exercises 5 Q11）：
  1. **停用**外洩的 KEK，產生新 KEK；
  2. 用新 KEK **重新包裝所有 DEK**；
  3. 如果有 DEK 已經落入攻擊者手中，那些 DEK 也要**換掉**，相關資料用新 DEK **重新加密**。
- （補充）記法：外洩哪一層，就換哪一層，再處理被它保護的下一層；只要下一層有可能已被拿走，就連下一層也要換。

## 5. Certificate 和 Web of Trust【小測：定義、目的】
- 網上交易靠一張**信任網**：A 為甚麼要信任一個從未交易過的 B？可以靠一個大家都認識的第三方替 B「擔保」（例如商會、信貸資料機構）。
- 有了 PKI，A 知道 B 的 public key 就可以 (1) 安全地發訊息給 B，(2) 確認 B 發來的訊息真的來自 B。
- **問題**：A 怎知道 B 給的 public key 真的是 B 的，而不是別人冒充的？分散式環境最常需要的信任，就是**可靠地把 public key 和身份綁在一起**。

### Certificate【小測】
- **定義**：電子版的「介紹信」。把**一條 public key 和一個用戶的身份綁在一起**，由 **certification authority（CA）簽署**，CA 以此擔保這個綁定正確。
- Certificate 用 **digital signature 和 hash function** 構成。
- **目的**：解決「這條 public key 是不是真的屬於這個人」的問題。

### 例子：公司總裁 X 替下屬 Y 發 certificate
1. Y 安全地把 {Y, K_Y}（身份和 public key）交給 X。
2. X 計算 hash：h({Y, K_Y})。
3. X 用自己的 private key 簽這個 hash，產生 {Y, K_Y, {h({Y, K_Y})}K_X⁻¹} → 這就是 **Y 的 certificate，由 X 簽署**。

### 驗證 certificate
Y 把 certificate 給 Z。Z 用 **X 的 public key** 驗證簽名，就知道：
- 這個 certificate 證明了 Y 和 K_Y 的綁定；
- 擔保人是 X；
- Y 和 K_Y 沒有被改過或損壞。
- **前提**：Z 要有一條**可信的 X 的 public key**。

Exercises 5 Q4／Q7：Bob 用 **CA 的 public key** 驗證 certificate 上的簽名。CA 的簽名證明 certificate 沒有被改過，而且 CA 確實擔保了身份和 public key 的綁定。

## 6. Chain of trust 和 CA【小測：是甚麼、目的】
- **Chain of trust**：X 有一張由 W 簽署的 certificate，現在 X 要替 Y 擔保，就簽一張 Y 的 certificate，再把自己的 certificate 附在後面 → 形成 W → X → Y 的信任鏈。
- **目的**：讓不直接認識簽署者的人，也能一級一級追溯到一個他信任的權威。理想情況下，鏈的根是一個無可置疑的權威，即 **root CA**。
- **Exercises 5 Q8**：網站的 certificate 由 CA A 簽，但用戶的電腦不直接信任 CA A → 檢查 CA A 的 certificate 是否由另一個用戶信任的 CA 簽署，可能經過幾個中間 certificate，**最終要追溯到一個受信任的 root CA**。
- **CA 的角色**【小測】：簽發 certificate，擔保身份和 public key 的綁定。CA 的權威來自**職位**而不是熟悉程度。實體世界的例子：公證人、人事主任、公司保安主任。
- 互聯網上的 root CA 例子：Verisign、SecureNet、Baltimore Technologies、Deutsche Telecom；香港認可的 CA：**香港郵政**、Digi-Sign、Hong Kong Certification Authority。

### X.509 certificate
X.509 是廣泛使用的 digital certificate 國際標準。X.509v3 的欄位：
1. Version
2. Serial number（同一簽發者內唯一）
3. Signature algorithm identifier（簽署用的演算法和參數）
4. Issuer's distinguished name（簽發者名稱）
5. Validity interval（有效期的開始和結束）
6. Subject's distinguished name（被擔保的一方）
7. Subject's public key info（演算法、參數、public key）
8. Issuer's unique ID（簽發者名稱被重用時用）
9. Subject's unique ID
10. Extensions
11. Signature（演算法、參數，以及第 1–10 欄的 hash 的簽名）

**驗證步驟**：取得簽發者的 public key（按第 3 欄的演算法）→ 驗證簽名（最後一欄）→ 重新計算 hash 並比較 → 檢查有效期。

## 7. 自我測驗
1. Alice 想發一份秘密文件給 Bob，同時讓 Bob 確定是她發的。要用哪些金鑰？
2. 硬碟被偷，但資料是 at rest 加密的，攻擊者還差甚麼才能讀資料？
3. 為甚麼 Google 換 KEK 不用重新加密所有資料？
4. 發現一條 KEK 外洩，但確定沒有任何 DEK 被拿走，要做甚麼？
5. 為甚麼 certificate 需要 CA 簽署，不能由用戶自己聲明「這是我的 public key」？

**答案**：
1. 用 **Alice 的 private key** 簽署（證明來源），再用 **Bob 的 public key** 加密（保護機密）。Bob 用自己的 private key 解密，用 Alice 的 public key 驗證簽名。
2. 金鑰。DEK 被 KEK 包裝，而 KEK 不能匯出 KMS。
3. 資料由 DEK 加密；換 KEK 只要用新 KEK 重新包裝 DEK，DEK 本身沒有變，資料不用重新加密。
4. 停用外洩的 KEK、產生新 KEK、用新 KEK 重新包裝所有 DEK。DEK 沒有外洩，就不用換 DEK，也不用重新加密資料。
5. 攻擊者也可以自稱是你並給出他的 public key（中間人攻擊）。要由受信任的第三方 CA 擔保身份和 public key 的綁定，別人用 CA 的 public key 驗證後才可以相信。

---

[← W05 ORCON、TCB、密碼學基礎](w05-orcon-tcb-crypto.md) ｜ [目錄](./)
