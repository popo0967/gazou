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

### 2.7 輪郭抽出と外接矩形の描画 (Contour Extraction & Bounding Box)
**用語:**
画像内の同じ色や明るさを持つ連続したピクセルの境界線を繋ぎ合わせる処理を「輪郭抽出」と呼びます。主に物体の形状認識、個数のカウント、サイズ計測などに利用されます。
*   **findContours:** 二値化画像から輪郭の座標データを抽出します。
*   **contourArea:** 抽出した輪郭に囲まれた領域の面積（ピクセル数）を計算します。微小なノイズを除去するフィルタリングによく使われます。
*   **boundingRect:** 抽出した輪郭をぴったり囲む、傾きのない長方形（外接矩形）の座標とサイズを計算します。

**アルゴリズム（処理の流れ）:**
1.  **前処理:** 画像をグレースケール化し、`cv2.threshold` で二値化することで、背景と物体（輪郭を抽出したい対象）を明確に分離します。
2.  **輪郭の検出:** `cv2.findContours` を用いて境界の座標群を取得します。
    *   `cv2.RETR_TREE`: 輪郭の中に別の輪郭がある場合（ドーナツ状の物体など）の親子関係（階層構造）をすべて保持するモードです。
    *   `cv2.CHAIN_APPROX_SIMPLE`: 直線上の連続する座標を省略し、端点のみを保持することでメモリを節約する近似手法です（例：四角形なら4つの頂点のみを保持）。
3.  **面積によるノイズ除去:** 検出されたすべての輪郭のうち、`cv2.contourArea` で計算した面積が1000ピクセル以下のものを、小さなゴミ（ノイズ）とみなしてリストから除外します。
4.  **外接矩形の計算と描画:** 残った有効な輪郭ごとに `cv2.boundingRect` を適用し、囲み枠の左上座標 $(x, y)$ と幅 $w$、高さ $h$ を取得して、元の画像に四角形（`cv2.rectangle`）を描画します。

**数式 (外接矩形の計算概念):**
ある輪郭を構成する点の集合を $C = \{(x_i, y_i)\}$ としたとき、外接矩形の左上座標 $(x, y)$、幅 $w$、高さ $h$ は次のように求められます。
$$x = \min_i (x_i), \quad y = \min_i (y_i)$$
$$w = \max_i (x_i) - x, \quad h = \max_i (y_i) - y$$

**Pythonコード:**
```python
import cv2
import matplotlib.pyplot as plt

img_bgr = cv2.imread('bolts.jpg')
img_bgr = cv2.resize(img_bgr, (500, 375))

# グレイスケールに変換する
img_gray = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2GRAY)

# 二値化(findContoursに渡すために必要です)
ret,img_bin = cv2.threshold(img_gray, 150, 255, cv2.THRESH_BINARY)

# 輪郭を抽出する
# contoursには個々の輪郭の形状(座標のリスト)が、
# hierarchyには輪郭の階層情報(親子関係)が入ります
contours, hierarchy = cv2.findContours(img_bin, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

# ノイズを取り除くために面積が一定以上の領域のみを残し残りを削除する
contours = list(filter(lambda x:cv2.contourArea(x) > 1000, contours))

# 領域の描画
result_img = cv2.drawContours(img_bgr.copy(), contours, -1, (0, 255, 0), 5)

# 個々の領域を囲む四角形(外接する矩形)を描画する
for c in contours:
    x,y,w,h = cv2.boundingRect(c)
    result_img = cv2.rectangle(result_img, (x,y), (x+w,y+h), (255, 0, 0), 4)

titles = ['Input Image', 'Grayscale', 'cv2.threshold', 'Result']
images = [img_bgr, img_gray, img_bin, result_img]

fig = plt.figure(figsize=(24,6))

for index, (title, img) in enumerate(zip(titles, images)):
    ax = fig.add_subplot(1,len(images),index+1)
    ax.title.set_text(title)
    ax.imshow(img, 'gray')
    ax.axis('off')
plt.show()
```


### 2.8 アフィン変換 (Affine Transformation)
**用語:**
画像の「平行移動」「回転」「拡大縮小」「せん断（スキュー）」を組み合わせた幾何学的な変換処理です。アフィン変換の最大の特徴は、**「変換前において平行だった直線は、変換後も平行が保たれる」**という性質を持っていることです。

**数式:**
変換前の座標を $(x, y)$、変換後の座標を $(x', y')$ とします。アフィン変換は、$2 \times 3$ の変換行列 $M$ を用いて、同次座標系との行列の積として以下のように表されます。

$$ \begin{pmatrix} x' \\ y' \end{pmatrix} = M \begin{pmatrix} x \\ y \\ 1 \end{pmatrix} = \begin{pmatrix} m_{11} & m_{12} & m_{13} \\ m_{21} & m_{22} & m_{23} \end{pmatrix} \begin{pmatrix} x \\ y \\ 1 \end{pmatrix} = \begin{pmatrix} m_{11}x + m_{12}y + m_{13} \\ m_{21}x + m_{22}y + m_{23} \end{pmatrix} $$

（※ $m_{13}, m_{23}$ は平行移動の成分を表します）

**Pythonコード:**
```python
import cv2
import numpy as np

# 画像サイズの取得
h, w = img_bgr.shape[:2]

# 1. 平行移動 (X方向に50, Y方向に20)
M_translate = np.float32([[1, 0, 50], [0, 1, 20]])
img_translated = cv2.warpAffine(img_bgr, M_translate, (w, h))

# 2. 回転と拡大縮小 (画像中心を軸に45度回転、スケール0.8倍)
center = (w // 2, h // 2)
M_rotate = cv2.getRotationMatrix2D(center, 45, 0.8)
img_rotated = cv2.warpAffine(img_bgr, M_rotate, (w, h))
```

### 2.9 画像の補間手法 (Interpolation Methods)
**用語:**
画像の拡大縮小や回転（アフィン変換など）を行った際、変換先の座標が整数にならない場合や、新しくピクセルを生成する必要がある場合に、**周囲のピクセルの値から新しいピクセルの色（画素値）を計算して推定する処理**を補間（Interpolation）と呼びます。

*   **最近傍補間 (Nearest Neighbor):** 最も近い位置にあるピクセルの値をそのまま採用します。計算速度は最速ですが、輪郭にギザギザ（ジャギー）が発生しやすく、ドット絵の拡大などに使われます。
*   **バイリニア補間 (Bilinear):** 周囲の4つのピクセルから、距離に応じた重み付け加重平均を計算します。処理速度と画質のバランスが良く、OpenCVの標準（デフォルト）手法です。
*   **バイキュービック補間 (Bicubic):** 周囲の16つのピクセルを使用し、3次関数を用いて滑らかに計算します。バイリニアよりも計算コストは高いですが、より高品質でシャープな結果が得られます。

また、INTER_Areaと呼ばれる(モアレ対策もある)

**数式 (最近傍とバイリニアの概念):**
変換先の座標 $(x, y)$（小数値を含む）に対する画素値 $f(x, y)$ の決定方法：
*   **最近傍補間:** 最も近い整数座標へ丸める。
    $$f(x, y) = I(\text{round}(x), \text{round}(y))$$
*   **バイリニア補間:** 周囲4点 $(x_1, y_1), (x_2, y_1), (x_1, y_2), (x_2, y_2)$ を用いて、距離（割合）で線形に重み付け（加重平均）する。

**Pythonコード:**
```python
import cv2

# 元画像のサイズ
h, w = img_bgr.shape[:2]

# 2倍に拡大するサイズ
new_size = (w * 2, h * 2)

# 1. 最近傍補間 (INTER_NEAREST) - 処理は早いがギザギザになる
img_nearest = cv2.resize(img_bgr, new_size, interpolation=cv2.INTER_NEAREST)

# 2. バイリニア補間 (INTER_LINEAR) - デフォルト。バランスが良い
img_bilinear = cv2.resize(img_bgr, new_size, interpolation=cv2.INTER_LINEAR)

# 3. バイキュービック補間 (INTER_CUBIC) - 処理は重いが滑らかで高画質
img_bicubic = cv2.resize(img_bgr, new_size, interpolation=cv2.INTER_CUBIC)
```


### 2.10 射影変換 (Perspective Transformation)
**用語:**
アフィン変換が「平行な線を平行なまま」変換するのに対し、射影変換（透視投影変換、ホモグラフィ変換とも呼ばれます）は「直線は直線のまま」ですが**「平行は保たれない」**変換です。
斜めから撮影したポスターや名刺を真正面から見たように補正したり、車載カメラ（ドライブレコーダー）の映像を真上から見下ろしたような「俯瞰画像（Bird's-eye view）」に変換したりする際によく用いられます。変換行列を計算するためには、変換前と変換後の**「4点の対応する座標」**が必要になります。

**数式:**
変換前の座標を $(x, y)$、変換後の座標を $(x', y')$ とします。射影変換は $3 \times 3$ の変換行列 $M$ と同次座標系を用いて、以下のように表されます。

$$ \begin{pmatrix} x'w \\ y'w \\ w \end{pmatrix} = \begin{pmatrix} m_{11} & m_{12} & m_{13} \\ m_{21} & m_{22} & m_{23} \\ m_{31} & m_{32} & m_{33} \end{pmatrix} \begin{pmatrix} x \\ y \\ 1 \end{pmatrix} $$

ここで計算された要素から、最終的な変換後の座標はそれぞれ $x' = \frac{x'w}{w}, \quad y' = \frac{y'w}{w}$ として求められます。

**Pythonコード:**
```python
import cv2
import numpy as np
from matplotlib import pyplot as plt

img = cv2.imread('drivecam.jpg')

# 変換前の4点の座標 (対象物の左上、右上、左下、右下などの対応する4点)
src_pts = np.array([(670, 680), (1130, 680), (130, 800), (1600, 800)], dtype=np.float32)

# 変換後の4点の座標 (ここでは500x500の正方形の四隅に割り当てて、真っ直ぐな矩形にする)
dst_pts = np.array([(0, 0), (500, 0), (0, 500), (500, 500)], dtype=np.float32)

# 4点の対応関係から、3x3の透視変換行列 M を計算する
M = cv2.getPerspectiveTransform(src_pts, dst_pts)

# 計算した変換行列 M を用いて画像に射影変換（ワープ）を適用する
dst_img = cv2.warpPerspective(img, M, (500, 500))

# 確認のため、元画像の指定した4点にマーカー(赤い×印)を描画する
cv2.drawMarker(img, (670, 680), (0,0,255), cv2.MARKER_TILTED_CROSS, markerSize=50, thickness=10)
cv2.drawMarker(img, (1130, 680), (0,0,255), cv2.MARKER_TILTED_CROSS, markerSize=50, thickness=10)
cv2.drawMarker(img, (130, 800), (0,0,255), cv2.MARKER_TILTED_CROSS, markerSize=50, thickness=10)
cv2.drawMarker(img, (1600, 800), (0,0,255), cv2.MARKER_TILTED_CROSS, markerSize=50, thickness=10)

# 元画像（マーカー付き）の表示
plt.imshow(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
plt.title("Original Image with Markers")
plt.show()

# 射影変換後（俯瞰）の画像の表示
plt.imshow(cv2.cvtColor(dst_img, cv2.COLOR_BGR2RGB))
plt.title("Perspective Transformed Image")
plt.show()
```


---

## 3. フィルタリングと高度な前処理

### 3.1 平滑化・ぼかし処理 (Smoothing / Blurring)
**用語:**
画像内の細かなノイズ（ざらつき）を減らしたり、細部をあえて潰して大きな構造だけを残したりする処理です。二値化やエッジ検出を行う前に適用することで、誤検出を大幅に防ぐことができます。
*   **ガウシアンフィルタ (Gaussian Blur):** 中心に近いピクセルほど重みを大きくする自然なぼかしです。
*   **メディアンフィルタ (Median Blur):** 周囲のピクセルの中央値（メディアン）を採用します。ごま塩ノイズ（突発的な白黒の点）の除去に非常に強力です。

**数式 (ガウシアンフィルタの重み):**
二次元ガウス関数を用いて、各ピクセルに対する重みを計算します（$\sigma$ は標準偏差）。

$$G(x, y) = \frac{1}{2\pi\sigma^2} \exp\left(-\frac{x^2 + y^2}{2\sigma^2}\right)$$

**Pythonコード:**
```python
import cv2

# 1. ガウシアンフィルタ (カーネルサイズは奇数で指定。ここでは 5x5)
img_gaussian = cv2.GaussianBlur(img_bgr, (5, 5), 0)

# 2. メディアンフィルタ (カーネルサイズを指定。ここでは 5)
img_median = cv2.medianBlur(img_bgr, 5)
```

### 3.2 エッジ検出 (Edge Detection)
**用語:**
画像の中で明るさ（輝度値）が急激に変化している境界線（エッジ）を見つけ出す処理です。物体の形状や輪郭といった「特徴」だけを抽出したい場合に用います。最も代表的なのが**Canny（キャニー）法**です。

**数式 (勾配の計算概念):**
エッジ検出の内部では、X方向とY方向の輝度の微分（勾配）$G_x, G_y$ を計算し、その大きさと方向を求めます。

$$|G| = \sqrt{G_x^2 + G_y^2}$$

**Pythonコード:**
```python
import cv2

# エッジ検出は通常グレースケール画像に対して行う
img_gray = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2GRAY)

# Canny法によるエッジ検出 (最小閾値と最大閾値を指定)
# ピクセルの勾配がmaxVal以上なら確実なエッジ、minVal以下ならエッジではないと判定
img_canny = cv2.Canny(img_gray, 100, 200)
```

### 3.3 モルフォロジー変換 (Morphological Transformations)
**用語:**
主に二値化画像に対して、形状を整形するための処理です。マスク処理の際に見られる「小さな穴あき」や「ノイズの斑点」を修復するために使います。
*   **Erosion (収縮):** 白い領域を削って細くします。小さな白点ノイズを消すのに有効です。
*   **Dilation (膨張):** 白い領域を太くします。かすれた線を繋いだり、Erosionで小さくなった領域を元に戻すのに使います。
*   **Opening / Closing:** 収縮と膨張を組み合わせた処理です（Closingは膨張→収縮の順に行い、領域内の黒い穴を埋めます）。

**Pythonコード:**
```python
import cv2
import numpy as np

# 操作の単位となるカーネル（構造要素）を定義 (ここでは 5x5 のすべて1の行列)
kernel = np.ones((5,5), np.uint8)

# 1. 収縮 (Erosion)
img_erosion = cv2.erode(img_bin, kernel, iterations=1)

# 2. 膨張 (Dilation)
img_dilation = cv2.dilate(img_bin, kernel, iterations=1)

# 3. オープニング (ノイズ除去) / クロージング (穴埋め)
img_opening = cv2.morphologyEx(img_bin, cv2.MORPH_OPEN, kernel)
img_closing = cv2.morphologyEx(img_bin, cv2.MORPH_CLOSE, kernel)
```

### 3.4 正規化と標準化 (Normalization & Standardization)
**用語:**
画像データを機械学習やディープラーニングのモデル（ニューラルネットワークなど）に入力する際、計算の安定化と学習速度の向上のために必須となる前処理です。
*   **正規化:** 画像の画素値（通常 0〜255）を `0.0 〜 1.0` などの範囲にスケール変換します。
*   **標準化:** 画像全体の画素値の平均を `0`、分散（標準偏差）を `1` になるように変換します。

**数式:**
画素値を $x$ としたとき、

*   Min-Max正規化: $$x' = \frac{x - \min(x)}{\max(x) - \min(x)}$$ (画像の場合は単純に255で割ることが多い)
*   標準化（Z-score正規化）: $$x' = \frac{x - \mu}{\sigma}$$ （$\mu$は平均、$\sigma$は標準偏差）

**Pythonコード:**
```python
import numpy as np

# 1. シンプルな正規化 (0.0 〜 1.0の範囲にスケーリング)
# float型に変換してから255で割る
img_normalized = img_bgr.astype(np.float32) / 255.0

# 2. 標準化 (平均0、標準偏差1にスケーリング)
mean = np.mean(img_bgr)
std = np.std(img_bgr)
# ゼロ除算を防ぐために分母に微小な値(1e-8)を足すのが一般的
img_standardized = (img_bgr - mean) / (std + 1e-8)
```