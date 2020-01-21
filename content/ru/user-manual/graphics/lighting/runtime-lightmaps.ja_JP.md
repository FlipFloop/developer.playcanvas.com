---
title: Runtime Lightmaps
template: usermanual-page.tmpl.html
position: 5
---

![Sponza][10]
*All the lighting in this scene is provided by lightmap textures*

ライトマップ生成は静的シーンの照明情報を事前に計算し、素材に適用されるテクスチャに格納する処理です。ライトソースや形状の多くが静的または環境に使用されている場合にシーンを照らす効率的な方法です。

## ランタイムのライトマップ生成 

PlayCanvasにはライトマップを生成する非常に便利な方法があります。Editorの標準的なライトコンポーネントを使用して、ライトマップをベークするために使用するライトと実行時にシーンを動的に照らすライトコンポーネントを選択します。ベークするために設定されたライトは、アプリケーションの起動時にシーンを照らすライトマップを生成するために使用されます。 

利点は次の通りです： 

* Lighting is not performed at **runtime**
* It is possible to use hundreds of static lights to light your scene
* Rendering lightmaps at runtime is, in many cases, faster than downloading many lightmap textures
* It is possible to mix static and dynamic lights in the Editor
* Rebaking can be performed at runtime
* Lightmaps are **HDR**
* Not only **Color** but **Direction** data can be baked as well, enabling some specularity on baked surfaces.

実行時のライトマップ生成を使用する欠点は、グローバルイルミネーション、アンビエントオクルージョンやその他の特殊なベーキングツールの高度な機能のをベーキングに現在対応していないということです。 

## ベーキング用にライトを設定

各ライトコンポーネントには、ライトマップのベーキングを有効にするための設定が含まれています。新しいライトはデフォルトで動的に設定されています。 

![ライトコンポーネントの設定][2]

**Lightmap: Bake**設定を有効にするとライトは範囲内にある全てのライトマップに含まれたモデルのライトマップをベークします。 

**Direction**オプションは、ライトがライト方向情報のベーキングに寄与するかどうかを指定します。シーン設定で**Color and Direction***ライトマップモードが選択されている場合、これは鏡面反射の結果に影響を与えます。

ライトの動作を変更する方法は他に二つあり、ライトが実行時に影響を与えるモデルを決定します。いずれかをtrueにするとライトは実行時に動作し、実行時の負荷に影響を与えます。 

**Affect Non-Baked** ボックスがtrueの場合、このライトはライトマップ**されていない**全てのモデルに影響を与えます。**Affect Baked** ボックスがtrueの場合、このライトはライトマップ**されている**全てのモデルに影響を与えます。 

照明の**Lightmap**と **Affect Baked** を両方trueにすることはできません。なぜなら、モデルにライトマップが生成されているのに、実行時にもライトが同じライティングを追加することになるからです。 

![ライトコンポーネントの影の設定][3]

ライトマップのライトの影の設定は動的ライトと同じですが、この場合はライトマップ生成時に一度影の計算が行われるのでライトマップのライトに影を有効にする方が軽くなります。[影][4]のセクションを参照してください。 

## ベーキング用にモデルを設定 

各モデルコンポーネントでライトマップを受信するには、ライトマップを有効にする必要があります。モデルコンポーネントのプロパティで**Lightmapped**オプションにチェックを入れてイトマップを有効にします。 

![モデルコンポーネントの設定][5]

**Shadows: Cast Lightmap** オプションはモデルがライトマップに影を落としているか否かを決定します。また、生成されたライトマップテクスチャの解像度を確認したり、UV1のエリアに乗数を適用して大きさを変更するためのオプションもあります。ライトマップのサイズ乗算については以下で説明します。 

## 一般的なライトの設定 

ご覧の通り、いくつかの組み合わせのライトの設定を使用することができます。これらの組み合わせには全てユースケースがあり、異なる組み合わせでライトを使用することにより、パフォーマンスと高品質なビジュアルのバランスをとることができます。

<table>
<tr>
    <th>Bake</th><th>Affect Non-Baked</th><th>Affect Baked</th><th style="width: 50%;">Description</th>
</tr>
<tr>
    <td class="centered">false</td><td class="centered">true</td><td class="centered">false</td><td>This is the default dynamic light. Affects all non-lightmapped models.</td>
</tr>
<tr>
    <td class="centered">true</td><td class="centered">false</td><td class="centered">false</td><td>This light generates lightmaps for lightmapped models and has no cost at runtime. Most static environmental lights could use this setting.</td>
</tr>
<tr>
    <td class="centered">true</td><td class="centered">true</td><td class="centered">false</td><td>This light generates lightmaps but also affects non-lightmapped models. It is useful if you have dynamic/moving entities that need to be lit with this light. For example, a prominent environment light that also should affect the player character.</td>
</tr>
<tr>
    <td class="centered">false</td><td class="centered">true</td><td class="centered">true</td><td>This light is a dynamic light which will affect both lightmapped and non-lightmapped models.</td>
</tr>
</table>

## ライトマッピングの設定

**Size Multiplier**はすべてのモデルコンポーネントに影響します。PlayCanvasは自動的にモデルに必要とされるライトマップの解像度を決定します。この値は、モデルのスケールおよび形状の面積の大きさに基づいて計算されます。モデルコンポーネントとのグローバル設定のサイズ乗数欄を修正することでこの計算に影響を与えることができます。 

1x1単位(メートル)の平面の例。グローバルサイズ乗数が16でモデルコンポーネントの乗数が2の場合、32x32のライトマップテクスチャサイズ (1sq/m * 16 * 2)が生成されます。つまり、１平方メートルあたり32x32ピクセルとなり、ピクセルサイズは約3cmです 。

**Max Resolution***は、メモリを節約するために生成されるライトマップの最大解像度に制限を設定します。

**Mode**では、どのデータをベークするかを指定したり、ピクセルからライトまでのみ色や方向を拡散することができます。方向データは、単純な鏡面反射をシミュレートするために使用されます。単一の方向しかベークすることができないので、複数のライトが重なった場合に複雑さを招きます。方向のベーキングは個々のライトに設定することもできます。

![グローバルライトマッピングの設定][6]

## 自動アンラップとUV1の生成 

ライトマップは、常にモデルアセットのUV座標(UV1)の第2のセットを使用して適用されます。最良の結果を得るためには、モデルをPlayCanvasにアップロードする前に、3Dコンテンツツールに第2のUVセットを追加することをお勧めします。[ライトマップに適したUVに関する推奨事項はこちらをご確認ください][9]。

便宜上、使用しているモデルがにUV1セットがない場合、PlayCanvasエディタは自動的にアンラップをしてモデルのためにUV1座標を生成することができます。 

![モデルコンポーネント：UV1 無し][7]

モデルのUV1マップが不足している場合は、ライトマッピングを有効にするときにモデルコンポーネントに警告が表示されます。 

![モデルアセット：パイプラインを自動でアンラップ][8]

警告を解決するには、モデルアセットを選択して**Pipeline**セクションを開きます。**Auto-Unwrap**ボタンをクリックして、プログレスバーが完了するのを待ちます。自動アンラップを実行するとモデルアセットが編集され、ソースからモデルを再インポートすると(例えば、新しいFBXをアップロード)あらかじめ計算されたUV1が削除され、アップロードされたモデルにUV1がない場合は、モデルを再度自動アンラップを行う必要があります。

**Padding**セクションでは、アンラップが発生した際のセクション間のスペースを決定します。ライトマップ上の不適切な場所にライトが表示されてしまう*"light bleeding"*が発生している場合、paddingを増やして軽減することができます。 

[1]: /images/user-manual/material-inspector/lightmap.jpg
[2]: /images/user-manual/lighting/lightmaps/editor-lightmap-bake.png
[3]: /images/user-manual/lighting/lightmaps/editor-light-shadows.png
[4]: /user-manual/graphics/lighting/shadows
[5]: /images/user-manual/lighting/lightmaps/model-settings.png
[6]: /images/user-manual/lighting/lightmaps/lightmapping-settings.png
[7]: /images/user-manual/lighting/lightmaps/model-uv1-missing.png
[8]: /images/user-manual/lighting/lightmaps/auto-unwrap.jpg
[9]: /user-manual/graphics/lighting/lightmapping/#uv-mapping
[10]: /images/user-manual/lighting/lightmaps/sponza.jpg
