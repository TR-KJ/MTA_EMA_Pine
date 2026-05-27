# v8.1α-D-S1-EMA Cross Strategy

## 概要

`v8.1α-D-S1-EMA Cross Strategy` は、  
1分足チャート上に表示した15分足MTAゾーンを基準に、1分足EMAクロスをトリガーとしてエントリーする初回ストラテジー版。

インジケーター版 `v8.1α-D-EMA Cross Trigger Test` の挙動確認後、  
同じトリガーロジックを使ってStrategy化したもの。

---

## 位置づけ

### 元インジケーター

- `v8.1α-A-MTA15 Base`
- `v8.1α-B-EMA Display Test`
- `v8.1α-C / C2.1-MTA Setup State`
- `v8.1α-D-EMA Cross Trigger Test`

### 今回のStrategy版

- `v8.1α-D-S1-EMA Cross Strategy`

---

## 目的

15分MTA Setupに対して、  
1分足EMAクロスをエントリートリガーにした場合の初回バックテストを行う。

目的は、別EMAトリガー案を増やす前に、  
まず1つの完成した「インジケーター → ストラテジー」検証の型を作ること。

---

## 基本コンセプト

### ロング

1. 15分押し安値MTAゾーンが有効
2. 1分足価格が15分押し安値MTAゾーンに接触
3. `L-SET` 発生
4. Setup有効期間内に1分足終値がEMAを上抜け
5. `L-TRIG` 発生
6. ロングエントリー
7. SLは15分押し安値MTAゾーン下限
8. TPはRR設定に基づいて計算

### ショート

1. 15分戻り高値MTAゾーンが有効
2. 1分足価格が15分戻り高値MTAゾーンに接触
3. `S-SET` 発生
4. Setup有効期間内に1分足終値がEMAを下抜け
5. `S-TRIG` 発生
6. ショートエントリー
7. SLは15分戻り高値MTAゾーン上限
8. TPはRR設定に基づいて計算

---

## 残しているロジック

以下は土台コードから変更しない。

- 15分MTA検出ロジック
- 15分MTAゾーン表示
- 15分MTA対応高値 / 安値表示
- 1分足上でのMTAゾーン接触判定
- MTAゾーン接触アラート
- EMA表示
- MTA接触後Setup状態
- EMAクロスTrigger表示

---

## 変更・追加したもの

インジケーター版 `v8.1α-D` から、以下を追加。

- `indicator()` から `strategy()` へ変更
- RR設定をinput化
- L/S TriggerをStrategyエントリーに接続
- `strategy.entry()` 追加
- `strategy.exit()` 追加
- SL/TP設定追加
- ポジション保有中は新規エントリーしない
- `pyramiding = 0`
- 手数料・スリッページ設定追加

---

## Strategy設定

```pine
strategy(
     "MTA v8.1α-D-S1-EMA Cross Strategy - 15m MTA on 1m",
     overlay = true,
     max_labels_count = 500,
     max_lines_count = 100,
     max_bars_back = 5000,
     initial_capital = 1000000,
     default_qty_type = strategy.percent_of_equity,
     default_qty_value = 100,
     commission_type = strategy.commission.percent,
     commission_value = 0.04,
     slippage = 2,
     pyramiding = 0,
     process_orders_on_close = false,
     calc_on_every_tick = false
)
