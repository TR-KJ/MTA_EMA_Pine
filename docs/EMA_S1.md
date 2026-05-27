# v8.1α-D-S1-EMA Cross Strategy

## 概要

`v8.1α-D-S1-EMA Cross Strategy` は、1分足チャート上に表示した15分足MTAゾーンを基準に、1分足EMAクロスをトリガーとしてエントリーする初回ストラテジー版。

インジケーター版 `v8.1α-D-EMA Cross Trigger Test` の挙動確認後、同じトリガーロジックを使ってStrategy化したもの。

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

15分MTA Setupに対して、1分足EMAクロスをエントリートリガーにした場合の初回バックテストを行う。

目的は、別EMAトリガー案を増やす前に、まず1つの完成した「インジケーター → ストラテジー」検証の型を作ること。

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
```

---

## エントリー条件

### Long Entry

```text
longSetupActive = true
かつ
close が EMA を上抜け
かつ
ポジション未保有
かつ
有効なSL幅がある
```

Pine条件イメージ：

```pine
closeCrossOverEma = ta.crossover(close, emaValue)

longEmaTrigger = false

if useEmaTrigger
    if longSetupActive
        if closeCrossOverEma
            longEmaTrigger := true
```

最終Entry条件：

```pine
canEnterLong = longEmaTrigger and strategy.position_size == 0 and not na(htfBullZoneBottom)
```

---

### Short Entry

```text
shortSetupActive = true
かつ
close が EMA を下抜け
かつ
ポジション未保有
かつ
有効なSL幅がある
```

Pine条件イメージ：

```pine
closeCrossUnderEma = ta.crossunder(close, emaValue)

shortEmaTrigger = false

if useEmaTrigger
    if shortSetupActive
        if closeCrossUnderEma
            shortEmaTrigger := true
```

最終Entry条件：

```pine
canEnterShort = shortEmaTrigger and strategy.position_size == 0 and not na(htfBearZoneTop)
```

---

## SL設定

### Long SL

15分押し安値MTAゾーン下限。

```pine
longStopPrice = htfBullZoneBottom
```

### Short SL

15分戻り高値MTAゾーン上限。

```pine
shortStopPrice = htfBearZoneTop
```

### バッファ

なし。

```text
SLバッファ：0
```

---

## TP設定

TPはRRによって計算する。

初期値：

```text
RR = 1.0
```

inputで変更可能。

```pine
rr = input.float(1.0, "RR", minval = 0.1, step = 0.1)
```

### Long TP

```pine
longRisk = close - longStopPrice
longTakeProfitPrice = close + longRisk * rr
```

### Short TP

```pine
shortRisk = shortStopPrice - close
shortTakeProfitPrice = close - shortRisk * rr
```

---

## Risk判定

Entry前に、SL幅が正しく成立しているか確認する。

### Long

```pine
validLongRisk = longRisk > 0
```

### Short

```pine
validShortRisk = shortRisk > 0
```

SL幅が成立しない場合はEntryしない。

---

## 注文仕様

### Long

```pine
if canEnterLong
    if validLongRisk
        strategy.entry("L", strategy.long)
        strategy.exit("L-Exit", from_entry = "L", stop = longStopPrice, limit = longTakeProfitPrice)
        longSetupActive := false
        longSetupStartBar := na
```

### Short

```pine
if canEnterShort
    if validShortRisk
        strategy.entry("S", strategy.short)
        strategy.exit("S-Exit", from_entry = "S", stop = shortStopPrice, limit = shortTakeProfitPrice)
        shortSetupActive := false
        shortSetupStartBar := na
```

---

## Setup管理

### L Setup開始

```text
15分押し安値MTAゾーンに1分足価格が接触
→ longSetupActive = true
```

### S Setup開始

```text
15分戻り高値MTAゾーンに1分足価格が接触
→ shortSetupActive = true
```

---

## Setupリセット条件

以下のいずれかでSetupを解除。

### 1. 反対側Setup開始

```text
L Setup中にS Setup開始
→ L Setup解除

S Setup中にL Setup開始
→ S Setup解除
```

### 2. MTA状態変化

```pine
if longSetupActive
    if htfTrendState != 1
        longSetupActive := false
        longSetupStartBar := na

if shortSetupActive
    if htfTrendState != -1
        shortSetupActive := false
        shortSetupStartBar := na
```

### 3. 有効期限切れ

```pine
setupExpireBars = input.int(30, "MTA Setup有効本数", minval = 1)
```

```pine
if longSetupActive
    if not na(longSetupStartBar)
        if bar_index - longSetupStartBar > setupExpireBars
            longSetupActive := false
            longSetupStartBar := na

if shortSetupActive
    if not na(shortSetupStartBar)
        if bar_index - shortSetupStartBar > setupExpireBars
            shortSetupActive := false
            shortSetupStartBar := na
```

### 4. Entry成立

```text
L Entry成立
→ longSetupActive = false

S Entry成立
→ shortSetupActive = false
```

---

## 1回のSetupで1回だけEntry

仕様：

```text
1回のL-SETにつき、L Entryは最大1回
1回のS-SETにつき、S Entryは最大1回
```

Trigger成立後、Setupを解除することで複数Entryを防止。

---

## ポジション保有中の扱い

```text
pyramiding = 0
ポジション保有中は新規Entryしない
反対シグナルでドテンしない
ExitはSL/TPのみ
```

つまり、

```text
ロング保有中にS Triggerが出てもショートEntryしない
ショート保有中にL Triggerが出てもロングEntryしない
```

---

## アラート

インジケーター版から継続。

### MTAゾーン接触アラート

```pine
alertcondition(useZoneTouchAlerts and bullZoneTouchOnce, title = "15m Long MTA Zone Touch", message = "1分足価格が15分押し安値MTAゾーンに接触しました")
alertcondition(useZoneTouchAlerts and bearZoneTouchOnce, title = "15m Short MTA Zone Touch", message = "1分足価格が15分戻り高値MTAゾーンに接触しました")
```

### EMA Triggerアラート

```pine
alertcondition(useEmaTriggerAlerts and longEmaTrigger, title = "Long EMA Cross Trigger", message = "L Setup後、1分足終値がEMAを上抜けました")
alertcondition(useEmaTriggerAlerts and shortEmaTrigger, title = "Short EMA Cross Trigger", message = "S Setup後、1分足終値がEMAを下抜けました")
```

---

## 表示

### 15分MTA関連

- 15分押し安値MTAゾーン
- 15分戻り高値MTAゾーン
- 15分MTA対応高値
- 15分MTA対応安値

### EMA

- 1分足EMA

### Setup

- `L-SET`
- `S-SET`

### Trigger

- `L-TRIG`
- `S-TRIG`

### データウィンドウ確認用

```pine
plot(showSetupStatePlot ? setupState : na, title = "AAA Setup State L=1 S=-1 Off=0", display = display.data_window)
plot(showSetupStatePlot ? (longSetupActive ? 1 : 0) : na, title = "AAA L Setup Active 1=ON", display = display.data_window)
plot(showSetupStatePlot ? (shortSetupActive ? -1 : 0) : na, title = "AAA S Setup Active -1=ON", display = display.data_window)
```

---

## 正直な限界・注意点

### 1. Entryは次足始値想定

`process_orders_on_close = false` のため、シグナル足でEntry注文を出し、次足始値で約定する想定。

### 2. TP計算はTrigger足close基準

今回のTPは、次足始値ではなくTrigger足のcloseを基準に計算している。

```text
Entry注文とExit注文を同時に出す安定性を優先した仕様。
ただし実際の約定価格とTP計算基準にはズレが出る可能性がある。
```

### 3. SLはTrigger時点のMTAゾーンを使用

Longは `htfBullZoneBottom`、Shortは `htfBearZoneTop` を使用。  
Entry後にMTAゾーンが変化しても、発注時点のSL/TPで管理する。

### 4. 15分MTAは上位足ロジック

`request.security()` を使って15分足MTAを1分足に展開している。

必ず以下を使用。

```pine
gaps = barmerge.gaps_off
lookahead = barmerge.lookahead_off
```

### 5. pine_checkだけでは不十分

`pine_check` / コンパイルが通っても、TradingView実機でのランタイムエラー確認が必要。

特に確認すること：

```text
赤い「!」が出ないか
注文が想定通り出るか
SL/TPが想定位置に出るか
古いスタディが残っていないか
必要なら削除→再追加する
```

---

## バックテスト時の注意

以下を最低限確認する。

```text
1. 取引数が十分にあるか
2. 勝率だけでなくPF、最大DD、平均損益を見る
3. RR変更時の変化を見る
4. EMA期間変更時の変化を見る
5. Setup有効本数変更時の変化を見る
6. ロング・ショート別成績を見る
7. 上昇相場・下降相場・レンジ相場で分けて見る
8. 過剰最適化しない
```

最低でも、

```text
50トレード以上
```

は見たい。

---

## 初期設定

```text
EMA期間：20
MTA Setup有効本数：30
RR：1.0
SLバッファ：なし
手数料：0.04%
スリッページ：2
pyramiding：0
初期資金：1,000,000
数量：100% of equity
```

---

## 今後の比較候補

D-S1を基準に、次のEMA Trigger案を比較する。

### E案

```text
EMAクロス + EMA傾きフィルター
```

条件：

```text
L：closeがEMA上抜け + EMA > EMA[1]
S：closeがEMA下抜け + EMA < EMA[1]
```

### F案

```text
短期EMA / 長期EMAクロス
```

条件：

```text
L：短期EMAが長期EMAを上抜け
S：短期EMAが長期EMAを下抜け
```

### G案

```text
EMAをTriggerではなくFilterとして使用
```

条件：

```text
L：別Trigger + close > EMA
S：別Trigger + close < EMA
```

---

## 保存名

```text
v8.1α-D-S1-EMA Cross Strategy
```

GitHub上の保存例：

```text
docs/strategies/v8.1a-D-S1-EMA-Cross-Strategy.md
```

または、

```text
docs/v8.1a-D-S1-EMA-Cross-Strategy.md
```

---

## コード本体の保存例

仕様書とは別に、Pineコード本体は以下のように分けて保存する。

```text
pine/v8.1a-D-S1-EMA-Cross-Strategy.pine
```
