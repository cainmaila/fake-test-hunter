<p align="center">
  <img src="assets/banner.png" alt="Fake Test Hunter: a hunter shatters a fake green checkmark with a glowing bolt" width="100%">
</p>

# Fake Test Hunter

[English](README.md) | **繁體中文** | [日本語](README.ja.md)

### 受夠了 AI 欺騙你?

你請 AI「補測試」。它回來時帶著一排綠勾、四十個新測試,還有一段信心滿滿的總結。

然後你故意把程式改壞。**所有測試照樣通過。**

這就是假測試:看起來像安全網,實際上什麼都沒保護。

```ts
// mock 回傳 42,然後測試斷言……42。
mockApi.getUser.mockResolvedValue({ id: 42 })
expect((await getUser()).id).toBe(42)

// 「它存在。」很好,那它正確嗎?
expect(result).toBeDefined()
```

AI 很會寫「會通過」的測試,卻不擅長寫「會失敗」的測試。這個倉庫給你的 agent 兩個 skill,專門補上這個缺口。

## 內容

| Skill | 功能 |
|-------|------|
| **`fake-test-hunter`** | 唯讀審查。同時讀測試和被測原始碼,逐一列出假測試:檔案、行號、類型、一句話理由。不抽樣、不湊數;沒抓到就直說沒抓到。 |
| **`test-squad`** | 平行派發專家子代理(各自在獨立 worktree)刪除、補強或新增測試。之後由**主線親自驗證**,不盲信專家的回報。 |

## 怎麼判定測試

測試**合格**,只要符合任一:

- **深**:輸入經過有意義的邏輯,斷言驗的是該邏輯的結果。
- **廣**:涵蓋極端值與錯誤路徑,不只 happy path。

測試是**假的**,只要符合任一:

- **理所當然**:不看程式也知道答案(斷言 mock 回傳值、`toBeDefined`、`assertIsNotNone`、型別系統已保證的事)。
- **不深不廣**。

## 為什麼 `test-squad` 不直接相信 AI

核心原則:**測試的價值,在於原始碼被改壞時它會 FAIL。**

所以每個新增或補強的測試都必須附上 mutation:改掉該測試聲稱驗證的邏輯,展示斷言失敗,還原後再展示通過。接著主線會:

1. 確認工作樹相對基準只多出測試檔改動。
2. 確認全套測試與靜態檢查的失敗集合,是基準失敗集合的子集。
3. 對每位專家抽樣,自己重做 mutation。只要有一個不符,該專家的工作全部重做。

如果新測試在現有程式碼上就失敗,`test-squad` **不會**改斷言去遷就。它會回報為疑似 bug,由你決定。

## 安裝

```bash
npx skills add cainmaila/fake-test-hunter
```

## 使用

```text
/fake-test-hunter                 # 審查專案會跑的全部測試
/fake-test-hunter src/auth        # 審查單一路徑

/test-squad src/auth              # 規劃、等你確認、派發、驗證
```

典型流程:先跑 hunter,把報告交給 squad,檢視計畫,核准,完成。

`test-squad` 會先問範圍,絕不全掃。被測原始碼有未提交改動時會停下來,派發前一定等你確認。

## 限制

- hunter 是 LLM 審查,不是 mutation testing 工具。它的清單是強線索,不是判決。
- mutation 無法觸及每一行程式碼,`test-squad` 會回報未涵蓋的部分。
- 專家越多,驗證成本越高。

## 授權

[MIT](LICENSE)
