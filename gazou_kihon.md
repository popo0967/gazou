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