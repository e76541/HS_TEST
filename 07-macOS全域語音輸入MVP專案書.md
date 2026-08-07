# 專案書：macOS 全域語音輸入 MVP

## 1. 專案目的

建立一個最小可運作的 macOS 全域語音輸入工具：

**使用者按下指定快捷鍵 → 開始說話 → 語音轉成文字 → 文字寫入目前使用中的輸入框。**

此專案取材自 Quill 的核心概念：將 Grok Build 既有的 Speech-to-Text 能力延伸到 macOS 其他應用程式，而不是限制在單一終端環境。

---

## 2. 專案核心 IPO

### 輸入 Input

- Control 單擊
- 使用者麥克風語音

### 處理 Process

1. 偵測 Control 單擊
2. 啟動麥克風錄音
3. 將音訊送入 Speech-to-Text
4. 接收轉錄文字
5. 找到目前作用中的文字輸入位置
6. 將文字寫入

### 輸出 Output

- 使用者說出的內容，以文字形式出現在目前作用中的應用程式輸入框。

---

## 3. 最小路徑

### 最小路徑 01：啟動錄音

#### 目的

讓使用者能用單一操作開始語音輸入。

#### IPO

**輸入**

Control 單擊

↓

**處理**

偵測全域鍵盤事件  
→ 判斷是否為單獨按下並放開 Control  
→ 啟動麥克風

↓

**輸出**

取得可供後續處理的音訊。

#### 本路徑主要學習點

- macOS 全域鍵盤事件
- 麥克風權限
- 音訊擷取

#### 完成判定

- 單擊 Control 可以開始錄音。
- Control+C 等組合鍵不應誤觸錄音。
- 能確認麥克風實際取得音訊。

原文的 Quill 同樣以單擊 Control 作為預設觸發，並特別區分單獨 Control 與 Control+C、Control+Space 等既有操作。

---

### 最小路徑 02：語音轉文字

#### 目的

把取得的音訊轉換成可使用的文字。

#### IPO

**輸入**

麥克風音訊

↓

**處理**

音訊送入 Grok Speech-to-Text  
→ 接收辨識結果  
→ 整理 transcript

↓

**輸出**

文字 transcript。

#### 本路徑主要學習點

- Streaming Speech-to-Text
- Grok Build Session／Authentication
- 音訊串流與文字回傳

#### 完成判定

說出：

`Hello World`

系統能取得：

`Hello World`

或語意相符的文字結果。

Quill 原始方案使用既有 Grok Build 登入狀態取得 Speech-to-Text 能力，並在錄音期間持續取得 transcript。

---

### 最小路徑 03：寫入目前輸入框

#### 目的

把 Speech-to-Text 的結果真正送到使用者正在工作的地方。

#### IPO

**輸入**

- transcript
- 目前作用中的應用程式／文字欄位

↓

**處理**

辨識目前焦點  
→ 找到可編輯文字欄位  
→ 使用 macOS Accessibility 寫入文字

↓

**輸出**

文字出現在目前使用中的輸入框。

#### 本路徑主要學習點

- macOS Accessibility API
- Focused Application / Focused Element
- 文字欄位寫入

#### 完成判定

在 Notes、瀏覽器或其他標準文字欄位中：

1. 點擊文字框。
2. 觸發語音輸入。
3. 說出內容。
4. 文字正確出現在該文字框。

Quill 在一般可編輯文字欄位中也是直接透過 macOS Accessibility 寫入，而非一律依靠剪貼簿貼上。

---

## 4. 三條路徑組合

完成三條最小路徑後：

**最小路徑 01**

`Control → 錄音`

＋

**最小路徑 02**

`音訊 → STT → Transcript`

＋

**最小路徑 03**

`Transcript → Accessibility → 文字欄位`

＝

### MVP 專案

`Control`  
→ `開始說話`  
→ `Speech-to-Text`  
→ `取得 Transcript`  
→ `找到目前輸入框`  
→ `寫入文字`

這時專案本身又形成一個更大的 IPO：

**輸入：人的語音**

→

**處理：錄音＋語音辨識＋系統輸入控制**

→

**輸出：應用程式中的文字**

這與原文最後濃縮的核心工作流一致：觸發、說話、讓文字進入指定位置。

---

## 5. MVP 驗收條件

第一版只需要證明：

> **我可以在另一個 macOS App 的文字框中，按 Control、說一句話，最後看到該句文字。**

符合以下條件即視為專案完成：

- Control 可以觸發錄音。
- 麥克風能取得聲音。
- STT 能產生 transcript。
- 能定位目前作用中的文字欄位。
- 能把 transcript 寫入。
- 整條流程可以連續跑通一次以上。

---

## 6. 第一版明確不做

以下內容雖然存在於 Quill 原文，但不納入目前 MVP：

- 浮動 Panel
- 即時 waveform
- 錄音計時器
- 自動靜音結束
- 「That's it」語音指令
- Open Grok
- 動態切換目的 App
- Click-to-insert
- Clipboard fallback
- 選取文字覆寫
- Transcript History
- 多螢幕支援
- Spaces 處理
- 自動錯誤分類
- 多種快捷鍵
- Start at Login

這些不是刪除，而是**尚未進入目前專案邊界**。

---

## 7. 後續擴展規則

MVP 跑通後，每一項新增能力重新建立自己的：

**輸入 → 處理 → 輸出**

最小路徑。

例如：

### 自動停止

`Transcript 停止更新`  
→ `計時 3 秒`  
→ `停止錄音並寫入`

### Open Grok

`語音包含「Open Grok」`  
→ `辨識控制語句＋啟動 Grok Build＋移除命令文字`  
→ `Grok 開啟並收到真正 Prompt`

### Clipboard Fallback

`Accessibility 無法寫入`  
→ `暫存剪貼簿＋貼上＋還原`  
→ `特殊 App 仍能收到文字`

因此後續不是把第一版專案無限制膨脹，而是：

**新需求 → 新最小路徑 → 驗證 → 再整合進較大的專案。**

---

## 8. 專案結構

```text
macOS 全域語音輸入 MVP
│
├─ LP01 啟動錄音
│  └─ Control → Audio
│
├─ LP02 語音轉文字
│  └─ Audio → STT → Transcript
│
├─ LP03 寫入文字
│  └─ Transcript → Accessibility → Text Field
│
└─ Integration
   └─ Control → Speak → Text appears
```

目前的專案邊界到此為止。

這版已經可以直接拿來驗證我們的「**最小路徑 × N → 專案**」模型；下一步最有價值的是再從這份專案書反推「每條最小路徑到底要從文本抽哪些知識」。

---

## 原文

[XFreeze：Quill](https://x.com/XFreeze/status/2085489360942567769)
