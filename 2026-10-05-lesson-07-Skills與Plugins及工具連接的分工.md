# 第 7 課｜Skills、Plugins 與工具連接的分工

- 日期：2026-10-05（台北時間）
- 時間：約 30 分鐘；程度：已有經驗的中低階使用者
- 接續：第 6 課「管理專案脈絡與可重用指示」（已在對話交付，當時未存檔）
- 學習目標：辨認任務缺少的是流程、資料存取或執行工具，並能設計沒有外部連接也能跟做的 Demo。

## 1. 30 分鐘安排

| 時間 | 活動 | 產出 |
|---|---|---|
| 0–6 分鐘 | 釐清四個名詞 | 能說明各自職責 |
| 6–11 分鐘 | 拆解錯誤報告流程 | 資料來源、整理規則、動作清單 |
| 11–21 分鐘 | 執行自製 Demo | 合併報告並保留證據 |
| 21–25 分鐘 | 閱讀驗證結果 | 區分空結果與讀取失敗 |
| 25–28 分鐘 | 需求分類練習 | 選出必要能力 |
| 28–30 分鐘 | 回顧與延伸 | 說出真實連接還缺什麼 |

## 2. 一個任務可能同時需要多種能力

官方文件的基本區分是：Skill 封裝任務指示與支援資源；Plugin 是可安裝的套件，可包含 Skills 與 MCP servers；MCP server 可提供工具，必要時還可提供自訂介面。[官方 Skills & Plugins](https://learn.chatgpt.com/docs/skills-and-plugins)

| 名詞 | 回答的問題 | 本課例子 |
|---|---|---|
| Skill | 這類工作按什麼流程做？ | 篩選未解決報告、合併重複、保留證據 |
| Plugin | 如何安裝及提供一組能力？ | 打包錯誤整理流程與資料工具 |
| MCP server | 如何向代理提供外部工具能力？ | 提供讀取錯誤報告的工具 |
| 工具 | 這次實際執行哪個操作？ | 讀取指定專案的一批報告 |

並非所有工具都透過 MCP 提供；內建執行環境等也能提供工具。也並非所有 Plugin 都一定含外部服務連接。

延續第 6 課：AGENTS.md 適合專案持續規則；Skill 適合可重複工作流程。兩者都不會僅靠文字就讓原本無法存取的服務突然可讀。

安裝 Plugin 與完成外部服務連接也是不同步驟。有的 Plugin 會在安裝或初次使用時要求連接帳戶，工具可用性仍受實際環境影響。[官方 Plugins](https://learn.chatgpt.com/docs/plugins)

## 3. Use case：整理今天的錯誤報告

官方 Automate bug triage 案例會從錯誤來源取得資料，合併重複報告，保留證據並建議後續處理；資料也可以來自貼上的內容或匯出檔。[官方案例](https://learn.chatgpt.com/use-cases/automation-bug-triage)

本課將它縮成一個可重現流程：

1. 取得一批報告，並確認是否成功讀取。
2. 只保留 open 報告。
3. 按來源提供的 fingerprint 合併。
4. 保留每筆報告的 id。
5. 按筆數排列，產出摘要。

**筆數只是本課的排序依據，不代表嚴重程度。**真正排優先級還需要影響人數、業務損失或故障範圍等證據。

### 哪一層負責什麼

| 工作 | 對應能力 |
|---|---|
| 讀取 Sentry 或 issue 系統 | 已連接服務的讀取工具 |
| 決定哪些報告要納入 | 提示詞或 Skill 中的流程規則 |
| 合併相同識別碼 | 可執行程式或其他適當工具 |
| 讓團隊安裝整套能力 | Plugin 的封裝與分發 |
| 建立 issue、發送訊息 | 額外的寫入操作，需要清楚的任務授權 |

今天沒有安裝 Plugin、建立 Skill、啟動 MCP server 或讀取真實帳戶。以下資料是固定範例；這讓我們能先驗證資料整理的邏輯。

## 4. Demo 的資料契約

固定範例包含四筆報告：

| id | fingerprint | state |
|---|---|---|
| R1 | checkout-zero-item | open |
| R2 | checkout-zero-item | open |
| R3 | coupon-invalid | open |
| R4 | old-display | closed |

本課約定：
- status 為 ok 時，items 是已通過欄位驗證的陣列。
- fingerprint 是來源已提供、可用於分組的識別碼；程式不自行推斷根因。
- status 不是 ok 時，表示資料不可用，不可報告「沒有錯誤」。
- 只整理資料，不修改輸入，也不執行外部寫入。
- 範例視為本次查詢的完整資料；真實工具是否有分頁、時間範圍或截斷，需要另外確認。

預期：
- checkout-zero-item：2 筆，證據 R1、R2。
- coupon-invalid：1 筆，證據 R3。
- R4 因已關閉而不納入。

## 5. 可複製提示詞

把上面的資料表或下一節 snapshot 物件一起提供：

```text
請整理這份錯誤報告快照。
資料是教學用固定範例，不需要連接任何外部服務。

流程：
先確認讀取狀態；不可用時明確回報，不能寫成沒有錯誤。
只保留 state 為 open 的項目。
依來源提供的 fingerprint 分組，保留所有報告 id 作為證據。
按報告數量由多到少排列；不要把數量當作嚴重程度。
只產出整理結果，不建立 issue 或發送訊息。

請提供最小 JavaScript 實作，驗證：
正常分組與 closed 排除、空資料、讀取失敗、輸入未修改。
說明哪些結果已執行，哪些真實工具連接尚未驗證。
```

這是一份流程提示詞，尚未封裝為 Skill。當它經多次使用仍有效，才適合考慮保存成可重用流程。

## 6. 完整可執行程式

操作：
1. 先手算四筆資料應變成幾個群組。
2. 請 Codex 執行下列 JavaScript，或在自己的 JavaScript 環境執行。
3. 若已有 Node.js，可另存 lesson07.js 後執行 node lesson07.js。
4. 比較成功的空陣列與 denied 狀態，觀察兩者輸出差異。

```javascript
function summarize(snapshot) {
  if (snapshot.status !== "ok") {
    return { status: "unavailable", message: "來源無法讀取；不能判定是否有錯誤" };
  }
  const groups = new Map();
  for (const item of snapshot.items) {
    if (item.state !== "open") continue;
    let group = groups.get(item.fingerprint);
    if (!group) {
      group = { fingerprint: item.fingerprint, count: 0, evidence: [] };
      groups.set(item.fingerprint, group);
    }
    group.count += 1;
    group.evidence.push(item.id);
  }
  const bugs = [...groups.values()].sort(
    (a, b) => b.count - a.count || a.fingerprint.localeCompare(b.fingerprint)
  );
  return { status: "ok", bugs };
}
const snapshot = {
  status: "ok",
  items: [
    {id:"R1", fingerprint:"checkout-zero-item", state:"open"},
    {id:"R2", fingerprint:"checkout-zero-item", state:"open"},
    {id:"R3", fingerprint:"coupon-invalid", state:"open"},
    {id:"R4", fingerprint:"old-display", state:"closed"}
  ]
};
const before = JSON.stringify(snapshot);
const result = summarize(snapshot);
if (JSON.stringify(result.bugs) !== JSON.stringify([
  {fingerprint:"checkout-zero-item", count:2, evidence:["R1","R2"]},
  {fingerprint:"coupon-invalid", count:1, evidence:["R3"]}
])) throw new Error("群組或證據不符");
if (summarize({status:"ok",items:[]}).bugs.length !== 0) throw new Error("空清單錯誤");
if (summarize({status:"denied"}).status !== "unavailable") throw new Error("來源錯誤被忽略");
if (JSON.stringify(snapshot) !== before) throw new Error("輸入遭修改");

console.log(JSON.stringify(result, null, 2));
console.log(summarize({status:"ok", items:[]}));
console.log(summarize({status:"denied"}));
console.log("四項檢查通過");
```

Map 用來依 fingerprint 儲存群組。evidence 陣列保存報告編號，方便回查。

### 本次實際驗證

核心程式與四項檢查已在 JavaScript 環境執行：

| 檢查 | 實際結果 |
|---|---|
| 分組、證據及 closed 排除 | 兩個群組；R1/R2 同組，R3 另一組，R4 排除 |
| 成功讀取但 items 為空 | status 為 ok，bugs 為空陣列 |
| status 為 denied | status 為 unavailable，未假裝資料為空 |
| 整理前後輸入比較 | 範例輸入未改動 |

預期摘要與實際分組相符：

| 群組 | 報告數 | 證據 |
|---|---:|---|
| checkout-zero-item | 2 | R1、R2 |
| coupon-invalid | 1 | R3 |

本次驗證不包含服務登入、真實 API 回傳、分頁、多來源合併或根因診斷。

## 7. 接上真實工具時要補哪些資訊

先確認資料在哪裡，再選擇可讀取它的能力。可用下面的任務說明：

```text
請從已連接的錯誤追蹤服務讀取指定專案、
指定時間範圍內的未解決報告。

請先確認來源是否可用，列出實際取得的資料範圍，
並說明是否還有分頁或未讀取資料。
依已確認的分組規則整理，附來源連結或報告編號。
如果無法存取，指出缺少的連接或來源，不推論沒有錯誤。
此次只產出摘要。
```

在實際使用時，要替換成明確的專案與時間範圍，並確認當前環境確實提供所需工具。此段僅示範寫法，本課沒有執行真實來源讀取。

最常見的四種情況：

| 現象 | 應先確認 |
|---|---|
| 知道流程，卻讀不到報告 | 是否有可用的來源工具與帳戶連接 |
| 能讀資料，摘要每次不同 | 是否缺少明確整理規則與輸出範例 |
| 只要整理貼上的資料 | 現有能力是否已足夠，不必增加連接 |
| 想讓團隊反覆使用整套流程 | 是否值得封裝成 Skill 或 Plugin |

## 8. 最佳實踐與常見錯誤

- **先驗證流程，再整合服務。**固定範例能幫助分辨是整理邏輯錯誤，還是資料連接出問題。
- **區分成功空結果與失敗。**服務無法讀取時，不能回覆「今天沒有錯誤」。
- **保留可追查證據。**分組後仍保存 id；不要只留下模糊摘要。
- **不要把模擬資料當成即時資料。**固定快照只能支持這批資料的結論。
- **不要用標題相似就斷言同一根因。**本課依來源的 fingerprint 分組，沒有推理根因。
- **明確區分讀取與寫入。**整理摘要與建立 issue 是兩個可分開授權的動作。
- **檢查取得範圍。**真實來源若只回傳第一頁，摘要不能宣稱覆蓋所有報告。

綜合應用：每週整理客戶回饋也同樣需要「資料取得、分類流程、證據保存、輸出格式」。資料來源換成匯出檔時，整理規則仍可重用。

## 9. 三分鐘練習

請判斷以下需求缺少哪種能力，並寫一句理由：

1. 已貼上 20 筆報告，希望依固定格式整理。
2. 想讀取私人倉庫的最新 issue。
3. 每次整理都忘記保留來源編號。
4. 希望團隊能安裝相同流程與連接工具。
5. 讀取結果顯示 permission denied，想知道是否沒有新問題。

### 參考答案與自我驗收

| 題目 | 參考答案 |
|---|---|
| 1 | 先用現有能力及明確提示詞；反覆使用時考慮 Skill |
| 2 | 需要能讀取該倉庫的工具、連接與存取權限 |
| 3 | 補足流程規則，並驗收每項是否有證據 |
| 4 | Plugin 可封裝相關能力，仍需驗證實際連接 |
| 5 | 無法判定；讀取失敗不等於沒有資料 |

- [ ] 沒把 Skill 當成服務帳戶授權。
- [ ] 沒把 Plugin 當成每次任務都必須安裝的東西。
- [ ] 能說出工具在執行哪個具體操作。
- [ ] 能分清資料錯誤與整理規則錯誤。
- [ ] 能說出本課只驗證固定資料，沒有讀真實服務。

## 10. 回顧與下堂預告

今天的判斷順序：**任務要什麼結果 → 資料在哪裡 → 有什麼可用工具 → 用什麼流程整理 → 如何驗收。**

下一堂：**設計可靠的工具輸入、輸出與失敗處理**。接續本課的空結果與讀取失敗，練習讓工具回傳更容易驗證，並避免把錯誤隱藏成正常答案。

## 來源與進度紀錄

- [Skills & Plugins](https://learn.chatgpt.com/docs/skills-and-plugins)：技能、插件與 MCP 工具的基本分工。
- [Plugins](https://learn.chatgpt.com/docs/plugins)：插件能力與服務連接。
- [Customization](https://learn.chatgpt.com/docs/customization/overview)：專案指示及可重用能力的定位。
- [Automate bug triage](https://learn.chatgpt.com/use-cases/automation-bug-triage)：整理錯誤來源、保留證據及分階段工作。
- 查閱日期：2026-10-05。
- 本課資料、程式、提示詞與練習為自製教學內容；沒有安裝插件或建立技能。
- 第 6 課已於 2026-10-02 在對話交付，本次接續第 7 課。
