<p align="right"><a href="../../../../../README.md">English</a> | <a>日本語</a></p>

# HLS C カーネル (alpha\_mix) の作成

設計仕様によれば、alpha\_mix カーネルには次の関数が含まれます。

* AXI Stream スレーブ インターフェイスを介したモノクロ (1 チャネル 8 ビット深度) デジタル クロック画像を受信
* AXI マスター インターフェイスを介してグローバル メモリから背景画像を読み出し
* デジタル クロック画像のサイズを指定したスケールに変更
* クロックの桁文字とクロック画像の背景レイヤーの色を指定
* クロック桁、クロック背景レイヤー、および背景画像 (不透明度設定あり) を混合
* AXI-Stream マスター インターフェイスを介して混合画像を出力

次のコード ブロックは、alpha\_mix カーネルの最上位関数の宣言部分です。ハードウェア AXI-Stream ポート (マスターまたはスレーブ) は hls::stream データ型で表記され、ハードウェア AXI マスター ポート (読み出しまたは書き込み) は ap\_uint (実際には配列ポインター) データ型で表記され、これらのカーネル引数は通常の int データ型で表記され、まとめて AXI スレーブ インターフェイスにマップされます。

~~~c++
void alpha_mix(hls::stream<ap_axiu<64, 0, 0, 0>> &time_img_input,   // time image input
               ap_uint<256> *bgr_img_input,                         // background image input
               hls::stream<ap_axiu<64, 0, 0, 0>> &mix_img_output,   // mixed image output
               int     time_img_rows_in,                            // input time image height
               int     time_img_cols_in,                            // input time image width
               int     time_img_rows_rsz,                           // resized time image height
               int     time_img_cols_rsz,                           // resized time image width
               int     time_img_pos_row,                            // resized time image position - Y
               int     time_img_pos_col,                            // resized time image position - X
               ap_uint<32> time_char_color,                         // [31:0] = [xRGB]
               ap_uint<32> time_bgr_color,                          // [31:0] = [xRGB]
               int     time_bgr_opacity,                            // time image background opacity，[7:0] used
               int     bgr_img_rows,                                // background image height
               int     bgr_img_cols                                 // background image width
)
~~~

次に、 alpha\_mix カーネルのサブ関数とデータフローの図を示します。

<div align="center"><img src="./images/alpha_mix_flow.png" alt="alpha_mix フロー" ></div>
この図の赤で塗りつぶされたサブ関数は Vitis ビジョン ライブラリのもので、青のサブ関数は手書きのものです。xf::cv::Mat は、OpenCV ソフトウェア ライブラリの cv::Mat に該当するもので、画像データの処理に非常に便利です。ハードウェア インプリメンテーションで画像を順番にピクセル レベルで処理する場合 (ピクセル データにランダムにアクセスする必要がない場合)、#pragma HLS stream を使用すると、コンパイラで xf::cv::Mat オブジェクトを配列にマップできます。これは、この alpha\_mix カーネルに該当します。Vitis ビジョン ライブラリの多くの関数は、入力および出力データとして xf::cv::Mat をサポートします。HLS C コードでは、for-loop を使用して、xf::cv::mat ストリーム内のデータを簡単に処理できます。入力および出力 xf::cv::Mat オブジェクトに適用されるアルファ ミキシング操作については、サブ関数ミキシング部分のソース コードを参照してください。

Vitis の v++ コマンドを使用して、HLS C ソースコードをカーネル ファイル (xo) にコンパイルします。たとえば、次のコマンドを使用して、xilinx\_u50\_gen3x16\_xdma\_201920\_3 プラットフォームを使用するハードウェア カーネルに alpha\_mix.c をコンパイルします。

~~~
v++ --platform xilinx_u50_gen3x16_xdma_201920_3                                                     \
    --target hw                                                                                     \
    --kernel alpha_mix                                                                              \
    --include ./include                                                                             \
    --advanced.prop kernel.alpha_mix.kernel_flags="-std=c++0x -D__SDSVHLS__ -DHLS_NO_XIL_FPO_LIB"   \
    --compile                                                                                       \
    --output alpha_mix.xo                                                                           \
    alpha_mix.c
~~~

---------------------------------------


<p align="center"><sup>Copyright&copy; 2020 Xilinx</sup></p>
<p align= center class="sphinxhide"><b><a href="../../../../README.md">メイン ページに戻る</a> &mdash; <a href="../../../README.md">ハードウェア アクセラレータ チュートリアルの初めに戻る</a></b></p></br>
<p align="center"><sup>この資料は 2021 年 2 月 8 日時点の表記バージョンの英語版を翻訳したもので、内容に相違が生じる場合には原文を優先します。資料によっては英語版の更新に対応していないものがあります。
日本語版は参考用としてご使用の上、最新情報につきましては、必ず最新英語版をご参照ください。</sup></p>
