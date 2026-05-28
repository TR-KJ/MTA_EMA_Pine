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
