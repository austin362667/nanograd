# 玩具神經網路 - 以三層為例

- 語言：Go
- 訓練資料集： MNIST 
- 測試資料集：手寫
- 損失函數: MSE
- 矩陣套件: gonum

原本打算不動腦直接模仿 Karpathy 的專案 [MicroGrad](https://github.com/karpathy/micrograd/tree/master/micrograd)

但 Python 和 Go 的語法實在差太多了...
最後還是需要仔細思考一下怎麼設計!

(我盡量使用 go 的標準函式庫，除了矩陣預算)

接著是一些數學算式(主要是為了練習 math latex)

### 執行方式：

`go build` 得到 binary 執行檔

輸入
`./nanograd -mnist train` 以訓練(約5~10 mins)

輸入
`./nanograd -mnist predict` 以預測(約5~10 secs)

## 從 input layer 到 hidden layer

$$
\begin{bmatrix} 
w_{11} & w_{21} \\
w_{21} & w_{22} \\
w_{31} & w_{23} \\
\end{bmatrix} 
\cdot
\begin{bmatrix} 
i_1 \\
i_2 \\
\end{bmatrix}
{=}
\begin{bmatrix} 
w_{11}*i_1 & w_{21}*i_2 \\
w_{21}*i_1 & w_{22}*i_2 \\
w_{31}*i_1 & w_{23}*i_2 \\
\end{bmatrix} 
$$

## 用線性回歸理解損失函數(的偏微分)

### MSE
$$
Error_{(m, b)}
{=}
\frac{1}{N}
\sum_{i=1}^N (y_i - (m*x_i+b))^2
$$

### Partial Derivative

- partial `m`

$$
\frac{\partial}{\partial m}
{=}
\frac{2}{N}
\sum_{i{=}1}^N {-}x_i*(y_i {-} (m*x_i{+}b))
$$

- partial `b`

$$
\frac{\partial}{\partial b}
{=}
\frac{2}{N}
\sum_{i{=}1}^N {-}(y_i {-} (m*x_i{+}b))
$$

## 從 hidden layer 到 output layer

$$
output_k
{=}
sigmoid(sum(\begin{bmatrix} 
w_{11}*o_{k1} \\
w_{21}*o_{k2} \\
w_{31}*o_{k3} \\
\end{bmatrix}))
$$

## 我們的損失函數(與梯度)

- 用這個計算錯誤程度的算法

$$
E_k {=} target_k {-} output_k
$$

- 求梯度

$$
\nabla w_{jk}
{=}
\frac{\partial E_k}{\partial o_k}
*
\frac{\partial o_k}{\partial {\sum}_k}
*
\frac{\partial {\sum}_k}{\partial w_{jk}}
$$

- 經過

$$
\frac{\partial E_k}{\partial o_k}
{=}
{-}(t_k - o_k)
$$

$$
\frac{\partial o_k}{\partial \sum_k}
{=}
o_k(1-o_k)
$$

$$
\frac{\partial \sum_k}{\partial w_{jk}}
{=}
o_j
$$

- 得

$$
\nabla w_{jk}
{=}
{-}(t_k - o_k)
{*}
o_k(1-o_k)
{*}
o_j
$$

## 反傳遞

根據前一個隱藏層的連接，輸出層的誤差是由來自隱藏層的誤差貢獻的。

換句話說，隱藏層的誤差組合形成了輸出層的誤差。

記得要 `Transpose` 權重矩陣

$$
\begin{bmatrix} 
w_{11} & w_{21} \\
w_{21} & w_{22} \\
w_{31} & w_{23} \\
\end{bmatrix} 
\cdot
\begin{bmatrix} 
e_1^{output} \\
e_2^{output} \\
\end{bmatrix} 
{=}
\begin{bmatrix} 
w_{11}*e_1^{output} & w_{21}*e_2^{output} \\
w_{21}*e_1^{output} & w_{22}*e_2^{output} \\
w_{31}*e_1^{output} & w_{23}*e_2^{output} \\
\end{bmatrix}
{=}
\begin{bmatrix} 
e_1^{hidden} \\
e_2^{hidden} \\
e_3^{hidden} \\
\end{bmatrix} 
$$

### 學習率

$$
\nabla w_{jk}
{=}
{-}lr {*} (t_k - o_k)
{*}
o_k(1-o_k)
{*}
o_j
$$

最後乘上學習率`learning rate (lr)`就完成啦！恭喜～

### 參考資料

- [梯度下降算法 by 陳鍾誠老師](https://gitlab.com/ccc110/ai/-/tree/master/07-neural/02-gradient)
- [Cross Entropy vs. MSE as Cost Function for Logistic Regression for Classification](https://www.youtube.com/watch?v=m0ZeT1EWjjI)
- [An Introduction to Gradient Descent and Linear Regression](https://spin.atomicobject.com/2014/06/24/gradient-descent-linear-regression/)

## log

```
0 &{{10 1 [0.9999581304431433 0.0003174777947214943 0.0008868458067127425 0.0013550321177239284 1.6422847081448215e-05 0.00264505839963317 0.00018725527055134117 0.00015294866564988295 1.5629050163170172e-06 0.0003687333613721123] 1} 10 1} 0.9999581304431433 0
1 &{{10 1 [1.5164738192743949e-06 0.9958626988562276 0.0017585169064314107 0.006094085387687636 0.00039575877696415335 0.00044657538855670087 0.00043058224318511705 0.003259800544038333 0.0011963014975710667 0.00154371685163943] 1} 10 1} 0.9958626988562276 1
2 &{{10 1 [0.022407340287417452 0.0024249469118764236 0.9688652637085451 0.001963842577318393 4.9673496333730424e-06 0.00011724000665576739 0.0001711447444465936 0.00033656163581201104 0.005488517327849982 5.938206748828088e-05] 1} 10 1} 0.9688652637085451 2
3 &{{10 1 [0.0020115220908991436 0.003142250276062238 0.0004736079986191409 0.9836117885316166 1.4773779656663114e-05 0.010701206374450675 1.7225821058116045e-06 5.545332943096245e-05 0.0008692469414799437 0.0952247564289989] 1} 10 1} 0.9836117885316166 3
4 &{{10 1 [2.9448402911084213e-05 0.0011436270058459569 5.855376881405744e-05 8.62564008482934e-05 0.9998206547260027 0.001823898566113614 0.00016749347961500852 0.00010442220314220119 4.531941428279753e-05 0.00028486792640040654] 1} 10 1} 0.9998206547260027 4
5 &{{10 1 [0.0019939515053200704 7.411077392659778e-05 6.878979625390618e-05 0.0008292132386637968 0.006367736226534001 0.9983346953078236 8.243744066440594e-07 1.8755206606090457e-05 0.0007876824094805287 0.00010135905684323706] 1} 10 1} 0.9983346953078236 5
6 &{{10 1 [0.00028534174937495883 0.00032494466125864704 0.0013808664017641618 0.00013922972361112322 0.0002609324640201032 0.0006613062219873224 0.9993898770969403 6.027931662790077e-05 0.0003200212158265398 8.790372315856775e-06] 1} 10 1} 0.9993898770969403 6
7 &{{10 1 [0.00034671463511118774 0.0007990273607095075 0.00026175475754723236 0.0023452563900765618 1.2907083252287712e-05 1.3126173233040828e-05 1.4161050423231651e-05 0.9996684832891315 2.310264584870699e-05 0.0003456901051119896] 1} 10 1} 0.9996684832891315 7
8 &{{10 1 [0.013843653526916062 3.329473704659782e-05 0.0025289891676587837 0.007223087635277064 0.00018649797783953443 0.006587650181145005 5.101197632591515e-05 0.00012471109309130865 0.9963255025976953 0.00017864164429323297] 1} 10 1} 0.9963255025976953 8
9 &{{10 1 [0.0015688027803710122 6.360950095541953e-05 0.0008511716555107429 0.00018240715424421833 0.068340605347173 0.0009624591011896311 0.0003171384174907803 9.772582376351171e-05 0.00046350413918701196 0.9987056302984713] 1} 10 1} 0.9987056302984713 9
0 &{{10 1 [0.9697916184835387 8.050945310605697e-06 0.0009421937449234223 5.115426427904517e-05 0.00010134147012533703 0.001335074729254559 0.0006673714972236093 6.210100669483478e-05 0.07997770718339733 0.0014781013938613369] 1} 10 1} 0.9697916184835387 0
1 &{{10 1 [1.814722566964608e-05 0.9953913062483866 0.0013200120939989776 0.00046684139684570074 0.00024343147951698648 0.0034703531256673535 0.0002310956787335039 0.0013956354428146858 0.0012013683167954071 0.00046559754214195334] 1} 10 1} 0.9953913062483866 1
2 &{{10 1 [0.002822518347481312 0.00042830188462441806 0.9992817380234906 0.0023968903976514483 1.732408158403192e-06 0.00043789493506129833 5.871235253550379e-05 9.41819901906614e-05 0.00018930110381966365 3.829890696713762e-06] 1} 10 1} 0.9992817380234906 2
3 &{{10 1 [0.0043750283495294275 0.0010623739970298462 0.0017611682755051868 0.9995594820623782 2.3110333266753173e-06 0.006075578904025186 2.324518384693678e-07 3.0291357489597175e-05 2.1795088875567833e-06 0.0012014357375117633] 1} 10 1} 0.9995594820623782 3
4 &{{10 1 [6.0719460106458046e-05 0.00031294554592298013 4.324855945091207e-05 0.000122386551744234 0.9990600714005563 0.0006573000660987443 0.00010345897330585912 0.0003340838521782208 0.0010660788115629716 0.00044144407566196393] 1} 10 1} 0.9990600714005563 4
5 &{{10 1 [0.00011949692044448685 6.667233471292506e-05 1.4201856329671423e-05 0.0003441198893147557 0.000669706405499444 0.9992604742712035 0.0006415628082965069 1.339409209046839e-05 0.052675038351412594 9.533370440304938e-06] 1} 10 1} 0.9992604742712035 5
6 &{{10 1 [0.00029864763572331674 0.00023939829269496585 0.0003497378535155227 8.611186695109016e-05 0.00039015687845050066 0.00043661096462864203 0.9970636875139909 7.283662215905025e-05 7.077046027868642e-05 3.160560125924273e-05] 1} 10 1} 0.9970636875139909 6
Time taken to check: 2.853937s
acc: 0.977200
 austin@AustindeMacBook-Air   ~/nanograd     main    2  INSERT  go build                                                     10202  0.72G   2.55    03:58:53 
 austin@AustindeMacBook-Air   ~/nanograd     main    2  INSERT  ./nanograd -mnist predict                                    10203  0.70G   2.62    03:59:25 
```