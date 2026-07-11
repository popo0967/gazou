# 1. 画像処理の基本演算と操作

## 1.1 AND演算 (Bitwise AND)
**用語:**
ビット単位の論理積を求める処理です。対応するピクセル（またはビット）が両方とも1（True）である場合のみ1を返します。主に、特定の領域のみを抽出する「マスキング処理」で利用されます。

**数式:**
画素位置 $(x, y)$ における入力画像を $I_1, I_2$、出力画像を $O$ とすると、論理積は以下のように表されます。
$$O(x, y) = I_1(x, y) \land I_2(x, y)$$

**Pythonコード:**
```python
import cv2

# 2つの画像のAND演算（画像は同じサイズであること）
result_and = cv2.bitwise_and(img1, img2)

# マスク画像を用いて特定領域だけを抽出する場合
result_masked = cv2.bitwise_and(img, img, mask=mask_img)
```

## 1.2 OR演算 (Bitwise OR)
**用語:**
ビット単位の論理和を求める処理です。対応するピクセル（またはビット）のどちらか一方が1（True）であれば1を返します。2つの画像の特徴や前景を統合する際などに用いられます。

**数式:**
論理和は以下のように表されます。
$$O(x, y) = I_1(x, y) \lor I_2(x, y)$$

**Pythonコード:**
```python
# 2つの画像のOR演算
result_or = cv2.bitwise_or(img1, img2)

# マスクを用いたOR演算
result_masked_or = cv2.bitwise_or(img, img, mask=mask_img)
```

## 1.3 addWeighted (画像の重み付き加算)
**用語:**
2つの画像を任意の割合（重み）で足し合わせる処理です。アルファブレンドやクロスフェード効果を作成する際に使用されます。各画素の足し算に係数を掛けることで、片方を透かして重ねるような表現が可能になります。

**数式:**
入力画像 $I_1, I_2$ に対するそれぞれの重みを $\alpha, \beta$、オフセット（定数）を $\gamma$ とすると、計算式は以下の通りです。
$$O(x, y) = \alpha \cdot I_1(x, y) + \beta \cdot I_2(x, y) + \gamma$$

**Pythonコード:**
```python
# img1を70%、img2を30%の割合で合成（ガンマ値=0）
alpha = 0.7
beta = 0.3
gamma = 0.0
result_blend = cv2.addWeighted(img1, alpha, img2, beta, gamma)
```

## 1.4 flip と rotate (反転と回転)
**用語:**
*   **flip:** 画像の反転処理です。X軸（上下）、Y軸（左右）、またはその両方に対して鏡像を作ります。データ拡張などで頻出します。
*   **rotate:** 画像の回転処理です。90度単位のシンプルな回転を高速に行うことができます。

**数式 (flipの例):**
画像の幅を $W$、高さを $H$ とした場合の座標変換です。
*   X軸反転（上下）: $(x', y') = (x, H - 1 - y)$
*   Y軸反転（左右）: $(x', y') = (W - 1 - x, y)$

**Pythonコード:**
```python
# flip (反転)
# flipCode: 0=上下反転, 1=左右反転, -1=上下左右反転
img_flip_h = cv2.flip(img, 1)  # 左右反転

# rotate (回転)
# cv2.ROTATE_90_CLOCKWISE (時計回りに90度)
# cv2.ROTATE_90_COUNTERCLOCKWISE (反時計回りに90度)
# cv2.ROTATE_180 (180度回転)
img_rotated = cv2.rotate(img, cv2.ROTATE_90_CLOCKWISE)
```

## 1.5 カラーチャンネル配列と画像の連結 (hconcat / vconcat)
**用語:**
*   **BGR配列:** OpenCVの `cv2.imread()` でカラー画像を読み込む際、データは一般的な「RGB」ではなく**「BGR（青・緑・赤）」**の順序で格納されます。そのため、matplotlibなどで表示する際はRGBへ変換する必要があります。
*   **hconcat (Horizontal Concatenation):** 複数の画像を水平（横）方向に連結します。
*   **vconcat (Vertical Concatenation):** 複数の画像を垂直（縦）方向に連結します。

**Pythonコード:**
```python
# BGR から RGB への変換 (matplotlibでの表示時などに必須)
img_rgb = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB)

# 画像の連結 (連結する画像は次元とサイズが合っている必要がある)
# hconcat: 横方向に並べる
img_h = cv2.hconcat([img1, img2])

# vconcat: 縦方向に並べる
img_v = cv2.vconcat([img1, img2])
```

---

## 2. 色空間の変換

### 2.1 グレースケール変換 (Grayscale Conversion)
**用語:**
カラー画像（RGBやBGR）から色情報を取り除き、明るさ（輝度）の情報のみを持つ白黒画像に変換する処理です。データ量（情報量）が1/3に減るため計算コストが下がり、輪郭抽出や物体検出（顔認識など）の前処理として非常によく使われます。

**数式:**
RGB値から輝度（$Y$）を求める一般的な計算式（NTSC加重平均法）は以下のようになります。
$$Y = 0.299 \times R + 0.587 \times G + 0.114 \times B$$

**Pythonコード:**
```python
import cv2

# BGR画像からグレースケール画像への変換
img_gray = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2GRAY)
```

### 2.2 HSV変換 (HSV Conversion)
**用語:**
RGB色空間を、人間の視覚・直感により近い表現に変換する処理です。照明の明るさが変化しても「色」を抽出しやすいため、特定の色をベースにしたマスク処理（特定の色だけを抜き出す処理）などで非常に重宝されます。
*   **H (Hue - 色相):** 色の種類（赤、青、緑など）。OpenCVでは通常 `0〜179` の範囲で表現されます。
*   **S (Saturation - 彩度):** 色の鮮やかさ。0に近いほどくすんだ色（グレーや白に近づく）になり、値が大きいほど純色になります（0〜255）。
*   **V (Value - 明度):** 色の明るさ。0が真っ黒で、値が大きいほど明るくなります（0〜255）。

**Pythonコード:**
```python
import cv2
import matplotlib.pyplot as plt
import numpy as np

img_bgr = cv2.imread('chocolates.jpg')
img_hsv = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2HSV)

h, s, v = cv2.split(img_hsv)

# 画像を大きめに表示する
plt.figure(figsize=(15,10))

# 2x3の領域に区切って合計6枚の画像をまとめて表示する
plt.subplot(2,3,1)
plt.title('Original (RGB)')
plt.imshow(cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB))

plt.subplot(2,3,2)
plt.title('Hue (H)')
plt.imshow(h, cmap='gray')

plt.subplot(2,3,3)
plt.title('Saturation (S)')
plt.imshow(s, cmap='gray')

plt.subplot(2,3,4)
plt.title('Value, Brightness (V)')
plt.imshow(v, cmap='gray')

# マスク画像(抜き出す部分が1, それ以外が0の画像）を作ります
# ここでは Hue が 80〜140かつ、Saturation が 70を超える部分を指定しています
mask = np.zeros(h.shape, dtype=np.uint8)
mask[(h > 80) & (h < 140) & (s > 70)] = 255

# そのままではマスク画像に穴が開くので closing 処理をすると綺麗なマスクになります
#41*41でうめて１があれば１で埋める、すべて1じゃないと消すを繰り返す
mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, np.ones((41,41),np.uint8))

# maskが1チャンネル, img_bgrが3チャンネルのためそのままでは AND が計算できません
# そのため cvtColor() でmask側を3チャンネルに変換しています。
mask_3ch = cv2.cvtColor(mask, cv2.COLOR_GRAY2RGB)

# bitwise_and() で AND をとり、MASKが0の部分が真っ黒になっている画像を出力します
result_img = cv2.bitwise_and(img_bgr, mask_3ch)

# マスク画像を表示
plt.subplot(2,3,5)
plt.title('mask')
plt.imshow(mask, cmap='gray')

# 最終的な出力を表示
plt.subplot(2,3,6)
plt.title('result')
plt.imshow(cv2.cvtColor(result_img, cv2.COLOR_BGR2RGB))

plt.show()
```
### 2.3 閾値処理 (Thresholding)
**用語:**
画像を白と黒の2階調（二値化）にするための最も基本的な処理です。特定のピクセルの輝度値が、あらかじめ設定した「閾値（Threshold）」よりも大きいか小さいかを判定し、白（255）か黒（0）に振り分けます。文字の抽出や物体の輪郭をはっきりさせる前処理として用いられます。

**数式:**
閾値を $T$、画素位置 $(x, y)$ における入力画像の画素値を $I(x, y)$、出力画像の画素値を $O(x, y)$、最大輝度値を $M$（通常は255）とすると、二値化の処理は以下のように表されます。
$$O(x, y) = \begin{cases} M & \text{if } I(x, y) > T \\ 0 & \text{otherwise} \end{cases}$$

**Pythonコード:**
```python
import cv2

# 閾値を127に設定し、超えるものを255(白)、それ以下を0(黒)にする
ret, thresh = cv2.threshold(img_gray, 127, 255, cv2.THRESH_BINARY)
```

### 2.4 大津の二値化 (Otsu's Method)
**用語:**
ヒストグラムの形状を解析し、自動的に「最適な閾値」を計算する手法です。画像を「背景」と「前景（物体）」の2つのクラスに分割したとき、2つのクラス間の画素値のばらつき（分離度）が最も大きくなるような閾値を数学的に導き出します。照明条件が安定しており、背景と前景の明暗が明確に分かれている画像で効果を発揮します。

**数式:**
ある閾値 $T$ で画像をクラス1とクラス2に分けたとします。それぞれのクラスに属する画素の割合（確率）を $\omega_1, \omega_2$、それぞれのクラスの平均輝度を $\mu_1, \mu_2$ とします。
このとき、2つのクラスがどれだけ離れているかを示す「クラス間分散 $\sigma_B^2$」は次のように表されます。
$$\sigma_B^2 = \omega_1 \omega_2 (\mu_1 - \mu_2)^2$$
大津の手法では、このクラス間分散が最大となる閾値 $T^*$ を最適な閾値とします。
$$T^* = \arg\max_{T} \sigma_B^2(T)$$

**Pythonコード:**
```python
# 閾値の引数は0にしておく。THRESH_OTSUを指定すると自動計算された閾値が ret に返る
ret, th_otsu = cv2.threshold(img_gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
```

### 2.5 トライアングルアルゴリズム (Triangle Algorithm)
**用語:**
大津の二値化と同じく自動で閾値を決定する手法の一つです。ヒストグラム上に「単一の大きな山（ピーク）」があり、その裾野にもう一つの小さな特徴が埋もれているような画像（例えば、大部分が暗い背景で、少しだけ明るい細胞が写っている顕微鏡画像など）で有効です。
ヒストグラムの最大ピーク点と、ヒストグラムの最遠端の点を直線で結び、その直線からヒストグラムの曲線までの垂直距離が最も長くなる輝度値を閾値とします。

**Pythonコード:**
```python
# THRESH_TRIANGLEを指定して自動二値化
ret, th_tri = cv2.threshold(img_gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_TRIANGLE)
```

### 2.6 適応的閾値処理 (Adaptive Thresholding)
**用語:**
画像全体に対して1つの固定された閾値を使う（グローバル閾値処理）のではなく、画像を小さな領域（ブロック）に分割し、**その領域ごとに周辺の画素値から局所的な閾値を計算して二値化する手法**です。
画像内に照明のムラがあったり、影が落ちていたりして、場所によって明るさが極端に異なる場合に非常に強力な効果を発揮します。

**数式:**
注目画素 $(x, y)$ の周辺領域（ブロックサイズ）内の平均値（またはガウシアン加重平均）を $\mu(x, y)$ とし、微調整のための定数を $C$ とします。このとき、場所 $(x, y)$ における固有の閾値 $T(x, y)$ は以下のように計算されます。
$$T(x, y) = \mu(x, y) - C$$
二値化の判定は、各画素に対してこの局所的な閾値を用いて行われます。
$$O(x, y) = \begin{cases} 255 & \text{if } I(x, y) > T(x, y) \\ 0 & \text{otherwise} \end{cases}$$

**Pythonコード:**
```python
# 近傍領域のサイズを11x11、微調整の定数Cを2とした例
# ガウシアン分布の重み付けを用いて局所閾値を計算 (ADAPTIVE_THRESH_GAUSSIAN_C)
th_adapt = cv2.adaptiveThreshold(
    img_gray, 
    255, 
    cv2.ADAPTIVE_THRESH_GAUSSIAN_C, 
    cv2.THRESH_BINARY, 
    11, 
    2
)
```