#目的:
フィルタとして使う

##
- G考え方

MTAゾーン接触

↓

Setup状態

↓

別Trigger成立

↓

その時点で close が EMA の有利側にいるか確認

↓

OKならEntry

Lなら close > EMA のときだけ許可
Sなら close < EMA のときだけ許可

##別トリガー:
いちばん自然なのは B5.2系のPivot Break Trigger

L：
15分押し安値MTAゾーン接触
→ Setup前後の基準Pivot Hを実体上抜け
→ その時点で close > EMA
→ L Trigger / Entry

S：
15分戻り高値MTAゾーン接触
→ Setup前後の基準Pivot Lを実体下抜け
→ その時点で close < EMA
→ S Trigger / Entry

なのでG案は、実質的には、
B5.2 Strategy + EMA Filter

##インジケータ

挙動OK

v8.1α-G-EMA Filter Trigger Indicator

##G案のストラテジー化

Gインジケーターのロジックをそのまま使い、決済・数量設定は B5.2 / D-S1.1 と同じ形式

##EMAをフィルタとして使用

v8.1α-G-S1.1-EMA Filter Strategy SL Mode Testのコードで、

MTA Zone        ：従来通り。L=HTF Bullゾーン下限、S=HTF Bearゾーン上限

Trigger Bar     ：Trigger成立足の安値/高値

Setup Touch Bar ：Setup接触足の安値/高値

MTA Zone Mid    ：MTAゾーン中央
