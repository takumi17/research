# A Survey of Quantization Methods for Efficient Neural Network Inference(効率的なニューラルネットワーク推論のための量子化手法に関する調査)  

## GENERAL HISTORY OF QUANTIZATION(量子化の一般的な歴史 )
GrayとNeuhoffは, 1998年までの量子化の歴史に関する非常に優れた調査を行っています【76】. この記事は非常に優れたものであり, 全体を読む価値がありますが, 読者の便宜のためにここでいくつかの主要なポイントを簡単に要約します. 量子化は, 大きな（しばしば連続的な）入力値の集合から小さな（しばしば有限の）出力値の集合にマッピングする方法であり, 長い歴史を持っています. 丸めや切り捨てが典型的な例です. 量子化は微積分の基礎に関連しており, 関連する方法は1800年代初期（およびそれ以前）に見られます. たとえば, 大規模な（1800年代初期の基準で）データ分析のための最小二乗法や関連する技術に関する初期の研究【225】です. 量子化に関する初期の研究は1867年にさかのぼり, 積分の計算を近似するために離散化が使用されました【206】. その後, 1897年にShappardが積分結果に対する丸め誤差の影響を調査しました【220】. 最近では, 量子化はデジタル信号処理において重要であり, 信号をデジタル形式で表現するプロセスには通常丸めが含まれ, 数値解析や有限精度の算術で実装される数値アルゴリズムの実装にも関連しています. 

1948年, デジタルコンピュータの出現時にShannonがコミュニケーションの数学的理論に関する画期的な論文を書いたとき【215】まで, 量子化の効果とその符号化理論での使用が正式に提示されることはありませんでした. 特に, Shannonはロスレス符号化理論において, 興味のある事象が一様でない確率を持つ場合, 同じビット数を使用することは無駄であると主張しました. 彼は, 事象の確率に基づいてビット数を変える方がより最適であると主張し, これは現在では可変レート量子化として知られる概念です. 特にハフマン符号化はこれに動機づけられています【109】. 1959年の後続の研究で【216】, Shannonは歪み率関数（符号化後の信号歪みの下限を提供する）やベクトル量子化の概念（セクションIV-Fでも簡単に説明）を紹介しました. この概念は拡張され, 実際の通信アプリケーションで実用的になりました【53, 55, 67, 208】. 当時の信号処理における量子化に関するその他の重要な歴史的研究には, パルス符号変調（PCM）概念を紹介した【188】や高解像度量子化の古典的な結果【14】があります. これらの問題に関する詳細な議論については, 【76】を参照してください. 

量子化は, 連続的な数学的量に関わる問題に対して数値近似を使用するアルゴリズムにおいて, やや異なる形で現れます. この分野も長い歴史を持っていますが, デジタルコンピュータの登場により新たな関心を集めました. 数値解析において重要な概念は, 適切に定義された問題というものでした. これは概略的に言えば, 解が存在し, その解が一意であり, その解が入力データに対して連続的に依存する場合に問題が適切に定義されているというものです. このような問題は, 時に良条件問題とも呼ばれます. しかし, 良条件問題であっても, 「理想的な」意味でその問題を「正確に」解く特定のアルゴリズムが, 丸め誤差や切り捨て誤差の特異性によって引き起こされる「ノイズ」の存在下で非常に悪い性能を発揮することが判明しました. これらの丸め誤差は, 実数を有限のビット数でしか表現できないことに関連しており, 例えばIEEE浮動小数点標準によって指定される量子化に起因します. そして, 切り捨て誤差は, 実際には反復アルゴリズムの有限回数の反復しか実行できないために生じます. 後者は, 「正確な算術」であっても重要であり, 連続数学のほとんどの問題は, 原理的に有限の基本操作の列で解決することはできないからです. しかし, 前者は量子化に関連しています. これらの問題は, アルゴリズムの数値安定性という概念につながりました. 

数値アルゴリズムを, 入力データ $x$ を「真の」解 $y$ にマッピングしようとする関数 $f $ と見なします. しかし, 丸め誤差と切り捨て誤差のために, アルゴリズムの出力は実際には別の $y^{*}$ になります. この場合, アルゴリズムの前方誤差は $\Delta y = y^{*} - y$ であり, 後方誤差は $f(x + \Delta x) = y^{*}$ を満たす最小の $\Delta x$ です. 前方誤差は, アルゴリズムによって出力されたものと正確な答えの違いを教えてくれ, 後方誤差は実際にアルゴリズムが正確に解いた入力データが何であったかを教えてくれます. アルゴリズムの前方誤差と後方誤差は, 問題の条件数によって関連付けられています. これらの問題に関する詳細な議論については【237】を参照してください. 

A. ニューラルネットにおける量子化  
これらのトピックに関しては数千の論文が書かれていることは間違いありませんが, 最近のニューラルネット（NN）量子化に関する研究はこれらの過去の研究とどう異なるのでしょうか？確かに, 最近提案された「新しいアルゴリズム」の多くは, 過去の文献に強い関連性があり, 一部は事実上再発見にすぎないものもあります. しかし, ニューラルネットには量子化の問題に特有の課題と機会があります. まず, ニューラルネットの推論とトレーニングはどちらも計算集約的です. そのため, 数値値の効率的な表現が特に重要です. 第二に, ほとんどの現在のニューラルネットモデルは大幅に過剰パラメータ化されているため, 精度に影響を与えずにビット精度を削減する余地があります. しかし, 非常に重要な違いの一つは, ニューラルネットが積極的な量子化や極端な離散化に非常に強いことです. ここでの新しい自由度は, 関与するパラメータの数に関連しており, 過剰パラメータ化されたモデルを扱っているということです. これは, 適切に定義された問題を解決しているかどうか, 前方誤差や後方誤差に関心があるかどうかなどに直接影響を与えます. 

量子化の最近の発展を推進しているニューラルネットアプリケーションでは, 単一の適切に定義された問題や良条件問題が解決されているわけではありません. 代わりに, 分類品質やパープレキシティなどに基づく前方誤差メトリックに関心があり, 過剰パラメータ化のおかげで, このメトリックを正確または概ね最適化する非常に異なるモデルが数多く存在します. そのため, 量子化されたモデルと元の非量子化モデルとの間に高い誤差や距離があっても, 非常に良い一般化性能を達成することが可能です. この追加の自由度は, 多くの古典的研究には存在せず, それらは主に信号をあまり変えない圧縮方法を見つけることに焦点を当てていたか, 「正確な」計算と「離散化された」計算の違いを強く制御する数値方法に関するものでした. この観察が, ニューラルネット量子化の新しい技術を研究する主な推進力となっています. 最後に, ニューラルネットモデルの階層構造は, 新たな次元を探求する機会を提供します. ニューラルネットの異なる層は, 損失関数に異なる影響を与えるため, 量子化に対する混合精度アプローチを動機付けています. 