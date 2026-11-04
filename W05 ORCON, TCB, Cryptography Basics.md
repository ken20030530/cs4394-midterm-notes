# W05 ORCON、Trusted Computing Base、密碼學基礎 詳細解說

> 來源：`W05 Week_5_Slides.pdf`、Exercises 4（答案，10/29 新版）、study guide。
> **【小測】** = study guide 點名要識；「（補充）」= 我加的解釋。

## 1. ORCON（Originator Controlled Access Control）【小測：是甚麼、目的】
- **要解決的問題**：資料**發出去之後**，怎樣繼續控制它怎樣被分享（Exercises 4 Q1）。
- 例子：農業部長寫了一份備忘錄給直屬下屬，任何人要再轉發都要她批准。控制權在**原作者**（originator）。

### 要求
Subject s 代表機構 X 把 object o 標為 ORCON。X 可以讓 o 給代表機構 Y 的 subject 看，但有兩個限制：
1. 沒有 X 的批准，o **不能**再給代表其他機構的 subject。
2. o 的**任何副本**都要帶著同樣的限制。

### 為甚麼 DAC 和 MAC 單獨都不行
- **DAC 不行**：擁有者（owner）可以隨意設定權限，而擁有者**不一定是原作者**。例如有人複製了一份，他就是副本的擁有者，可以改權限 → 第 2 個要求做不到。
- **MAC 單獨也不行**：MAC 的類別由中央管理員統一管理。如果 A 想把文件給 B 看，但不准 B 再轉給 C，就要為每種分享情況開新類別 → **category explosion（類別爆炸）**，也很難臨時授權沒有預計到的人。ORCON 需要在地方層面、彈性地（最好分散地）控制。

### ORCON = MAC 和 DAC 的混合
- **MAC 的部分**：object 的擁有者**不能**改它的存取限制；複製時，限制會跟著副本走（擁有者控制不到）。
- **DAC 的部分**：**原作者**可以按每個 subject、每個 object 彈性地修改限制，支援進一步分享。

## 2. DRM（Digital Rights Management）：ORCON 的實際應用
- **目的**（Exercises 4 Q2）：控制、管理和限制數碼媒體、應用程式和文件的存取，即**控制傳播**；在商業環境落實原作者控制，防止盜版。
- **6 個元素**（用下載電影做例子）：

| 元素 | 意思 | 電影例子 |
|---|---|---|
| **Content** | 被保護的資料 | 電影 |
| **License** | 描述內容可以怎樣使用的 token | 把播放權綁定到這一份下載的副本 |
| **Grant** | License 的一部分，給某些實體特定權限，可以附條件 | 只能在某些設備、某個地區播放 |
| **Issuer** | 發 license 的實體 | 電影公司 |
| **Principal** | License 適用於誰 | 下載電影的用戶 |
| **Device** | 用來觀看內容的機制 | 播放設備，負責管理 license、principal 和副本 |

- **3 個條件**：
  1. 系統要**控制內容的使用方式**。只是加密再把金鑰給合資格的觀眾**不夠**，因為用戶可以隨便把金鑰分享出去。
  2. 限制規則要**跟著內容**，不是跟著用戶。
  3. 不論怎樣傳播、傳給誰，控制和規則在內容的整個生命期都要**一直有效**。

### Microsoft PlayReady
- 用 rights expression language 表達條件：觀看期限、license 有效期、限制複製／轉移／轉換、地區限制、可用性限制（例如廣播期間不能播）。
- 設定：內容用 **AES** 加密；金鑰交給 license server，加密內容交給 distribution server。
- 播放：用戶下載內容並申請 license → license server 驗證用戶，成功就造 license 送回 → 用戶端檢查限制，允許的話就用 license 內的金鑰解密。
- 硬件保護：PlayReady 整合到 **Trusted Execution Environment（TEE）**。

### Apple FairPlay
- 授權電腦：iTunes 產生一個全球唯一的號碼送去 Apple server，加入該用戶的授權名單（**最多 5 部**）。
- 取得內容：內容用 **AES master key** 加密；master key 再用 iTunes 隨機產生的 **user key** 鎖住；user key 加密後存在 Apple server 和 iTunes。
- 播放：iTunes 解密 user key → 用 user key 解密 master key → 用 master key 解密內容（**不用聯絡 Apple server**）。
- 授權新電腦：Apple server 把所有 user key 送去那部電腦。取消授權：刪除本機所有 user key，並通知 Apple 從名單刪除號碼。
- 把內容抄去另一部電腦：沒有 user key（user key 不會跟著抄）→ 解不了密。

## 3. Trusted Computing Base（TCB）【小測：定義、reference monitor 的 3 個特性】
- **TCB**：電腦中**負責執行安全政策**的所有硬件、firmware 和軟件的集合。
- **越小越好**：減少實作出錯的機會，也較容易仔細驗證。
- TCB 以外的錯誤不會幫攻擊者違反安全政策。
- **Unix 工作站的 TCB 最少包括**：kernel 和所有 device driver；所有以 root 權限運行的程序；所有 root 擁有並設了 set-user-ID 的程式；用來建造以上各項的程式庫和開發工具；CPU；儲存裝置和它們的 firmware；檔案伺服器和它們網絡連線的完整性。其中**任何一個**有漏洞，都可以繞過整個 Unix 存取控制。

### Reference monitor【小測】
- 一個抽象機器：**調解（mediate）所有 subject 對所有 object 的存取，而且不能被繞過**。
- **Security kernel** 是 reference monitor 在某個硬件上的實作，例如 Solaris、Red Hat Linux、Mac OS X。
- 現在也有在更高層次（Java／C# 虛擬機用型別檢查分隔區域）或更低層次（Xen、VMware 等 hypervisor）提供同樣功能的。
- **3 個特性**：
  1. **Complete**：調解**每一次**存取，不能被繞過。
  2. **Isolated**：不能被系統內其他實體修改，受到保護。
  3. **Verifiable**：只做它被設計要做的事，不會因惡意行為或程式員出錯而被繞過，可以被測試證明正確。

### Protection rings（保護環）
- 越裡面的環越受信任；要由外環進入內環，要通過額外的安全機制。保護環是 TCB 概念的一部分。
  - **Layer 0**：最受信任，OS kernel，以特權模式運行
  - **Layer 1**：OS 的非特權部分
  - **Layer 2**：I/O driver、底層操作和工具
  - **Layer 3**：應用程式，用戶平時接觸的層

### 網絡上的 rings of trust
- 把網絡上的主機分環。決定時問：主機是否在有保安的機房？有沒有一般用戶帳戶？是否在遙遠的分部（較不可信）？是否依賴來自互聯網的資料？是否提供關鍵服務、停機會影響多少人？
- **規則**：每部主機信任**更內環**的主機；**不信任**更外環的主機；可以信任同一環的主機；同一環分成不同子網的話，不同子網之間不互相信任。
- Slides 提示：這和 **Biba Strict Integrity** 非常相似。（補充）內環 = 高 integrity，只接受來自同級或更高的東西，即 no read down。

## 4. 密碼學基本概念
- **加密**的目的：令竊聽者拿到的訊息沒有用處。
- 記號：**C = E(M)**、**M = D(C)**，所以 M = D(E(M))。用金鑰時：C = E(M, Ke)、M = D(C, Kd)。
  - Ke = Kd → **對稱（symmetric）**；Ke ≠ Kd → **非對稱（asymmetric）**。
  - 也常寫成 {M}K。
- 解密等於從一條「有系統地加了雜訊」的通道還原訊息。**冗餘（redundancy）是安全加密的敵人**，因為它給攻擊者線索。
- 密碼分析員（cryptanalyst）可能想：破解一則訊息；找出加密訊息的模式；不破解也推斷出一些意思；推斷金鑰；找出實作、環境或使用方法的弱點；找出演算法本身的弱點。
- 他們的工具：密文、已知的演算法、截獲的明文、已知或懷疑出現在密文的資料、數學和統計方法、語言的特性、電腦，以及聰明和運氣。

## 5. 5 種攻擊：按攻擊者有甚麼資料分類【小測：要會分類】

| 攻擊 | 攻擊者有甚麼 |
|---|---|
| **Ciphertext-only** | 只有密文 |
| **Known plaintext** | 有一些「密文＋對應明文」的配對 |
| **Chosen plaintext** | 可以**揀**明文，叫系統加密給他看 |
| **Adaptive chosen plaintext** | Chosen plaintext，而且可以按之前的結果調整下一個要揀的明文 |
| **Chosen ciphertext** | 可以揀密文，叫系統解密給他看 |

Exercises 4 Q8：(a) 只有幾則密文 → ciphertext-only；(b) 有密文也知道部分對應明文 → known plaintext；(c) 可以揀訊息再拿到密文 → chosen plaintext。**最有機會成功的是 chosen plaintext**，因為攻擊者掌握最多控制和資料。
（補充）判斷技巧：看攻擊者能不能「揀」；能揀明文 → chosen plaintext，能揀密文 → chosen ciphertext；只是剛好知道一些配對 → known plaintext。

### 資料怎樣縮小搜尋範圍（xyy 的思考實驗）
密文是 "xyy"，3 個英文字母。
1. 只知道是 substitution cipher → 每個位置可以是任何字母：26³ = **17,576** 種。
2. 再知道是 **simple** substitution（一對一）→ 明文的形式一定是「abb」，而且 a ≠ b：26 × 25 = **650** 種，縮小約 27 倍。
3. 再知道明文是英文單字 → 符合「abb」的 3 字母單字約 **40** 個（例如 see、too、all），比第 1 步縮小約 439 倍。
- 結論：密碼分析就是**用所有資料減少對明文的不確定**。

## 6. 完美密碼和 One-Time Pad【小測：是甚麼、為甚麼完美】
- **完美密碼（perfect cipher）**：知道演算法和密文，也**完全不能縮小**搜尋範圍。即攻擊者有沒有密文，猜中明文的機會都一樣。
- **Shannon 證明**：完美密碼需要**和明文數量一樣多的金鑰**，而且金鑰要隨機選。
- **One-time pad**（Miller 1882；Vernam 和 Mauborgne 1917）：金鑰**和明文一樣長**、**只用一次**，和明文做 **XOR**。明文、密文和金鑰的空間都相同。
- **為甚麼完美**：例如截獲 3-bit 密文 `101`。對任何一個 3-bit 明文 P，都有金鑰 K = P XOR 101 能產生這個密文，所以 8 個明文都同樣可能，完全不能縮小範圍。
  （補充）例如：明文 000 用金鑰 101 → 101；明文 111 用金鑰 010 → 101，兩個都說得通。
- **問題在實務，不在理論**：**key distribution problem**。如果雙方已經有安全的通道，又何需金鑰？如果沒有，又怎樣安全地把金鑰交給對方？而且金鑰要和訊息一樣長。
- **近似做法**：用 **PRNG（偽隨機數產生器）** 由一個秘密的 seed 產生金鑰流，接收方用同一個 seed 產生同一條金鑰流。這啟發了現代的 stream cipher：AES-CTR、AES-OFB、AES-CFB、RC4。

## 7. 好的加密：breakable vs strong【小測：substitution／transposition】
- 好加密的準則：數學基礎穩固；經專家分析確認可靠；經得起時間考驗。
- **Breakable**：只要時間和資料足夠，分析員能還原明文。大部分演算法都 breakable，因為可以逐個金鑰試。但**可以破解不等於實際上做得到**；分析員還要能**認出**甚麼時候成功，所以通常需要明文和密文的配對。
- **Strong**：沒有比 **brute force（逐個金鑰試）** 明顯更快的分析方法。大部分 strong 的演算法仍然 breakable；金鑰空間越大，搜尋越久。現代對稱加密建議用 **256-bit** 金鑰。

### 兩個基本構件
- **Substitution（替換）**：每個符號換成另一個符號，例如 AES 的 SubBytes 用一個固定的 8-bit 查找表替換每個 byte。
- **Transposition（換位）**：重新排列符號的次序，例如 AES 的 ShiftRows 重排 16-byte 陣列。
- 幾乎所有現代商業對稱密碼都結合兩者。它們達到兩個效果：
  - **Confusion（混淆）**：轉換明文的資訊，令截獲者難以直接抽出 → **substitution 擅長**
  - **Diffusion（擴散）**：把明文一小部分的資訊擴散到密文的很大範圍 → **transposition 擅長**

## 8. 對稱 vs 非對稱加密【小測：金鑰、要不要交換金鑰】

| | **對稱（symmetric）** | **非對稱（asymmetric）** |
|---|---|---|
| 別名 | private key、secret key | public key |
| 金鑰 | 加密和解密用**同一條**金鑰 | 每人一對：**public key**（公開）和 **private key**（自己保管） |
| 要不要交換秘密金鑰 | **要**，每對用戶要共享一條秘密金鑰 | **不用**交換秘密金鑰，但要能取得對方的 public key |
| n 個用戶兩兩通訊要幾多條金鑰 | **n(n−1)/2**，即 O(n²) | **2n**，即 O(n)（有些演算法加密和簽署要分開兩對） |
| 金鑰特性 | 隨機的 k-bit 字串，容易產生，沒有特別性質 | 有特殊結構（例如大質數），產生成本高 |
| 優點（Exercises 4 Q4） | **較快、較有效率**，適合大量資料 | **解決 key distribution problem**，不用預先共享秘密金鑰 |

- 兩種金鑰的長度不能直接比較：256-bit 對稱金鑰的強度大約等於 2048-bit 的 public key。
- **非對稱加密怎樣解決 key distribution**：要秘密地發訊息給 B，就用 **B 的 public key** 加密，只有持有 **B 的 private key** 的 B 能解密。Public key 可以公開派發，不用秘密通道。
- **Exercises 4 Q6**：Alice → Bob：Alice 用 PK_B 加密，Bob 用 SK_B 解密；Bob → Alice：Bob 用 PK_A 加密，Alice 用 SK_A 解密。不用交換秘密金鑰，但雙方要取得對方的 public key（仍然要 key distribution，只是派發的是公開的 public key）。
- **Exercises 4 Q5**：100 個用戶。(a) 每人各自連公司資料庫 → **100** 條（每人一條，和伺服器共享）；(b) 兩兩互傳訊息 → 100 × 99 / 2 = **4,950** 條。
- 兩大挑戰：**key distribution**（怎樣把金鑰交給需要的人）、**key management**（金鑰很多時怎樣保管並在需要時取用）。

## 9. AES（現代對稱加密）
- 現代對稱加密是 **block cipher**（固定大小的區塊）＋ **rounds**（重複對「狀態」做相似運算，即 iterated block cipher），用便宜的機器運算，專為快速處理大量資料而設計。
- 1995 年 NIST 公開徵選新標準：不保密、公開、全球免版稅、128-bit 區塊、支援 128／192／256-bit 金鑰。15 個候選中，選了比利時研究員 Rijmen 和 Daemen 的 **Rijndael** 作為 **AES**（slides 寫荷蘭）。
- 經大量分析未發現缺陷（不代表沒有）；金鑰長度和 round 數可以加。
- **Modes of operation**：區塊模式 AES-CBC；金鑰流模式 AES-CFB、AES-CTR、AES-OFB。AES 只是構件，**安全性要靠正確的 mode**。
- 形式上：Enc 是**隨機化**（非確定性）的，密文比明文長一點（因為 padding）；Dec 是確定性的；不知道金鑰的人，除了長度外看密文**學不到任何關於明文的東西**。

## 10. Cryptographic hash function【小測：3 個性質】
- 把任意長度的文字變成一個**固定長度**的小值，叫 **message digest**。
- **3 個性質**：
  1. **Preimage resistance**：給你一個 hash 值，很難造出一段有這個 hash 的文字。
  2. **Weak collision resistance**：給你一段文字，很難改它而 hash 不變。（補充）即很難找到另一段 hash 相同的文字，又叫 second-preimage resistance。
  3. **Strong collision resistance**：很難找到**任何兩段**不同的訊息有相同的 hash。
- 用途：**integrity**，不是 confidentiality。例如法律文件庫要確認取出的副本和儲存的一模一樣。Hash 把檔案的每個 byte 綁在一起，任何改動都會被發現，像給檔案加封條。
- 常用：**SHA-2**（SHA-224／256／384／512）、**SHA-3**。**MD4、MD5、SHA-1 已出現碰撞，不再建議**用於需要碰撞抵抗的地方。
- 注意：Exercises 4 Q9 官方答案對 strong 和 weak collision resistance 的解釋寫得較簡略（「沒有兩個輸入有相同 hash」「一個訊息只有一個 hash」），考試時建議用 slides 的定義。

## 11. MAC 和 Authenticated Encryption【小測】
### MAC（Message Authentication Code）
- 雙方**預先共享一條秘密金鑰**，用來驗證訊息。
- MAC 函數：用 k-bit 金鑰 K 和訊息 M 算出一個 n-bit 的 **tag**；不知道 K 的人猜中 tag 的機會不比 2⁻ⁿ 好。
- **用途**：確認訊息**沒有被改過**（integrity），而且**來自持有金鑰的一方**（authentication）。
- 常用：基於 block cipher 的 **CMAC-AES**；基於 hash 的 **HMAC**-SHA-2 等。

### Authenticated Encryption（AE）
- 一般加密只保證 **confidentiality**，MAC 只保證 **integrity**；但在不可信的通道（TLS、IPsec、SSH）兩樣都要，而且要兩樣同時做對**很難**：沒有安全證明就不要自己發明方案。
- **AE 的 2 個特點**【小測】：
  1. 提供**密文的 integrity**（當然也有加密帶來的機密性）。
  2. **解密時會拒絕**任何不是用同一條金鑰、經加密函數產生的密文。
- 常用：**AES-GCM**、AES-CCM、AES-EAX、AES-OCB。
- 一種做法：用 key derivation function 由一條金鑰衍生兩條新金鑰，**先加密**明文，**再把密文的 MAC 附在後面**（encrypt-then-MAC）。
- Exercises 4 Q7：一般加密（例如 AES）→ C；AE → **C + I**。

## 12. 總表：各種密碼學工具提供甚麼、要不要金鑰【小測】

| 工具 | Confidentiality | Integrity | Authentication | Non-repudiation | 要金鑰？ |
|---|---|---|---|---|---|
| 加密（對稱或非對稱） | ✔ | | | | **要** |
| Hash | | ✔ | | | **不用** |
| MAC | | ✔ | ✔ | ✘（雙方共享同一條金鑰，任何一方都能造 tag） | **要**（共享秘密金鑰） |
| Digital signature（W06） | | ✔ | ✔ | ✔ | **要**（private key 簽，public key 驗） |
| Authenticated encryption | ✔ | ✔ | ✔（補充：能通過驗證，代表由持有金鑰的人產生） | | **要** |

Exercises 4 Q10：要金鑰的是**加密、MAC、digital signature**；**hash 不用**。
（補充）MAC 不能提供 non-repudiation 這點 slides 沒有寫，是一般說法：雙方都有同一條金鑰，第三者分不出 tag 是誰造的。

## 13. 自我測驗
1. 攻擊者偷到一部會自動加密的裝置，可以輸入任何訊息看輸出。這是哪種攻擊？
2. 為甚麼「把電影加密、把金鑰給付費用戶」不符合 DRM 的條件？
3. 50 個用戶，對稱和非對稱加密兩兩通訊各要幾多條金鑰？
4. 下載軟件時網站附上 SHA-256 值，這保護了甚麼？能否證明軟件來自真正的開發者？
5. Reference monitor 的 3 個特性是甚麼？

**答案**：
1. Chosen plaintext（可以揀明文）。
2. 違反條件 1：用戶可以隨便把金鑰分享出去，系統控制不到內容的使用。
3. 對稱 50 × 49 / 2 = **1,225**；非對稱 2 × 50 = **100**。
4. 只保護 **integrity**（下載的檔案沒有被改過）。如果攻擊者同時改了網站上的 hash 值，就檢查不出來；hash 不用金鑰，**不能證明來源**，要證明來源要用 digital signature。
5. Complete（調解每次存取）、isolated（不能被修改）、verifiable（可以驗證正確）。
