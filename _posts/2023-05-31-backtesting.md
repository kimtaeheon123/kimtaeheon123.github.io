---
layout: single
title:  "ATR지표를 이용해 백테스팅하는 방법"
categories: 
    - project1 
tags: [python, backtesting, ATR]
typora-root-url: ../
---

Python의 Backtesting 라이브러리를 사용하지 않고 ATR지표를 이용해 백테스트하는 방법에 대해 알아본다.

**Backtesting 준비**

open, high, low, close와 ATR 값을 가진 시계열 데이터를 불러온다.
```python
df1 = pd.read_csv("/content/drive/MyDrive/candledata/BTCUSDT, 15.csv")
```

트레이딩뷰에서 가져온 BTCUSDT 15분봉 데이터를 사용했다. 

![image-20230531171049914](/images/$(filename)/image-20230531171049914.png)

양봉일때 매수하여 종가 + 1.5*atr값에 먼저 도달할 경우 수익 실현, 종가 - 1.5*atr가격에 먼저 도달할 경우 손실을 입는 구조이다. 

```python
close = list(df1['close'])
open = list(df1['open'])
length = len(df1)

Bullish = [0]*length

for i in range(len(df1)):
  if close[i]-open[i] > 0:
    Bullish[i] = 1
  else:
    Bullish[i] = 0

df1['Bullish'] = Bullish
dfpl_Bullish = df1[(df1['Bullish']==1)]
dfpl_Bullish['index'] = dfpl_Bullish.index
```

위 코드를 통해서 dfpl_Bullish 이름의 데이터 프레임 안에 양봉으로 마감한 경우만 따로 추출했다. 

```python
dfpl = df1
length_B = len(dfpl_Bullish)
result = [0]*length_B
profit = [0]*length_B
k = 1.5
r=25
for l in range(length_B):
  atr = dfpl.loc[dfpl_Bullish.iloc[l]['index']]['ATR']
  Loss = dfpl.loc[dfpl_Bullish.iloc[l]['index']]['close'] -k*atr
  Gain = dfpl.loc[dfpl_Bullish.iloc[l]['index']]['close'] +k*atr

  for i in range(1,r):
    if dfpl_Bullish.iloc[l]['index']+i >= len(dfpl):
      break
    if (Gain <= dfpl.loc[dfpl_Bullish.iloc[l]['index']+i]['high']) and (Loss >= dfpl.loc[dfpl_Bullish.iloc[l]['index']+i]['low']):
      result[l] = 3
      profit[l] = 0
      break
    elif (Gain <= dfpl.loc[dfpl_Bullish.iloc[l]['index']+i]['high']):
      result[l] = 1
      profit[l] = k*atr/dfpl.loc[dfpl_Bullish.iloc[l]['index']]['close']*100 
      break   
    elif (Loss >= dfpl.loc[dfpl_Bullish.iloc[l]['index']+i]['low']):
      result[l] = 2
      profit[l] = -k*atr/dfpl.loc[dfpl_Bullish.iloc[l]['index']]['close']*100 
      break    

dfpl_Bullish['result'] = result
dfpl_Bullish['profit'] = profit

print("k:",k) #몇배의 atr를 곱할 것인지
print("r:",r)#몇번째 캔들까지 볼것인지
print(dfpl_Bullish['result'].value_counts())
print("합계 수익률: ",dfpl_Bullish['profit'].sum())
```

result가 1인 경우 설정한 Loss값보다 Gain값에 먼저 도달하여 수익, 2인 경우는 반대로 손실이며 3인 경우는 한 캔들안에서 Gain과 Loss에 둘다 도달했다는 것이다. 이때는 더 낮은 시간 프레임에서 백테스팅해야된다. result가 0인 경우는 포지션 진입 이후 25번째 캔들까지 위 과정을 수행하는데 그 기간동안 목표가격에 도달하지 못했다는 뜻이다.

![image-20230601115538789](/images/$(filename)/image-20230601115538789.png)
