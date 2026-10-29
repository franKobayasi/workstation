---
id: 8796e4
description: 把模糊的非功能需求轉成可度量、可排序的品質屬性場景與 utility tree 的方法。
---

# 品質屬性分析

功能需求決定系統做什麼，品質屬性決定系統長什麼樣；選微服務、加快取、做多活，都是為了滿足某個品質屬性。品質屬性分析把「系統要快、要穩」這類描述，變成可度量、可驗證、可排序的場景。

## 蒐集候選屬性

從三個來源萃取：

1. **明示需求**：PRD 或需求文件中寫出的非功能需求。
2. **隱含需求**：沒人寫但必然存在的要求；安全性、可維護性與成本幾乎永遠隱含存在。
3. **業務反推**：從商業目標倒推，例如大促對應彈性伸縮，金融業務對應稽核性。

參考清單，挑相關的使用：

| 類別 | 屬性 |
|---|---|
| 執行期 | Performance、Scalability、Availability、Reliability、Elasticity |
| 安全合規 | Security、Privacy、Auditability、Compliance |
| 演進期 | Modifiability、Maintainability、Testability、Deployability、Extensibility |
| 營運 | Observability、Operability、Recoverability（RTO/RPO）、Cost efficiency |
| 使用 | Usability、Accessibility、Interoperability、Portability |

## 挑選驅動屬性

- 只挑 3 到 7 個驅動屬性。支援每個屬性都有代價，屬性之間也常互相衝突。
- 強迫排序，最多標 1 到 2 個為最高優先；全部都重要等於都不重要。
- 每個驅動屬性寫一句它為什麼驅動這個系統，連回業務目標。

## 場景化

每個驅動屬性至少寫 1 到 2 個場景，採六要素格式：

| 要素 | 說明 | 範例 |
|---|---|---|
| Source | 誰或什麼觸發 | 外部金流服務 |
| Stimulus | 發生什麼事 | 回應逾時率升至 50% |
| Artifact | 系統的哪個部分 | 訂單結帳服務 |
| Environment | 在什麼狀態下 | 大促尖峰時段 |
| Response | 系統應如何反應 | 降級為非同步扣款並通知使用者 |
| Response Measure | 可驗證的標準 | 結帳成功率 ≥ 99%，P99 延遲 < 2s |

沒有 Response Measure 的場景不合格。度量不確定時，給出合理的預設值並標明「假設，待確認」，不留空。

## Utility tree

以「品質屬性 → 細分關注點 → 場景」的樹狀結構整理，每個場景標兩個維度：

- **業務價值**（H/M/L）：做不到會多痛。
- **技術風險**（H/M/L）：以目前的設計達成有多難。

```
Availability
├── 對外服務不中斷 → 金流故障降級場景（H, H）
└── 資料不遺失     → 節點故障零遺失場景（H, M）
```

兩個維度皆為 H 的場景是設計優先滿足、審查優先走查的對象。

## 產出

1. 驅動屬性表：屬性、為什麼驅動本系統、優先級。
2. 場景清單：ID、屬性、六要素、（業務價值, 技術風險）。
3. Utility tree：文字樹或 Mermaid mindmap。
4. 待確認的假設清單，特別是自行推定的度量數值。

可自動驗證的場景（延遲門檻、依賴規則）建議轉為 fitness function 納入 CI，讓品質屬性被持續看守。

## 反模式

- 寫出無法驗證的形容詞，例如快、穩、安全。
- 不敢排序，全部列為最高優先。
- 把功能當品質，例如「支援匯出報表」是功能需求。
- 憑空訂 SLA：99.999% 的成本是 99.9% 的數倍，先問業務真正需要幾個 9。
- 忽略隱含屬性：沒人提到 Security 不代表不需要。
