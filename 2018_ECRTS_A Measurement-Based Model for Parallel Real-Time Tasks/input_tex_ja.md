
## Abstract
マルチプロセッサ・スケジューリングの連合パラダイムの下では、プロセッサのセットは各リアルタイム・タスクの排他的使用のために予約される。タスクが（セーフティクリティカルなシステムで典型的なように）非常に保守的にキャラクタライズされる場合、タスクのほとんどの呼び出しは、ワーストケースのキャラクタライズをはるかに下回る計算要求である可能性が高く、タスクの実行時動作のワーストケースのキャラクタライズを仮定して割り当てられたよりもはるかに少ないプロセッサで正しくスケジューリングされる可能性がある。すべてのプロセッサが必要になるタイミングをランタイム中に安全に決定できれば、残りの時間、不要なプロセッサは低エネルギーの「スリープ」モードでアイドル状態にするか、バックグラウンドで非リアルタイムの作業を実行するために使用することができる。本稿では、並列化可能なリアルタイム・タスクを表現するためのモデルを提案する。このモデルは、タスクによって表現されるコードの内部構造に関する詳細な知識を必要としない。むしろ、異なる条件下でコードを繰り返し実行し、実行時間を測定することによって得られるいくつかのパラメータによって、各タスクを特徴付ける。

## 1. Introduction
スケジューリング理論は、リアルタイムシステムの解析に関係する。リアルタイムシステムのマルチプロセッサやマルチコアの実装が普及するにつれて、リアルタイム作業負荷を表現するスケジューリング理論で使用されるモデルは、これらの作業負荷内に存在する可能性のある並列性を明らかにすることができることが望まれている。この必要性から、フォークジョインモデル[1、2]、スポラディックDAGタスクモデル[3]（教科書的な説明は[4、21章]を参照）、マルチDAGモデル[5]、条件付きDAGタスク_モデル[6、7]などの正式なタスクモデルが生まれた。これらのモデルはそれぞれ、モデル化されるコードの内部構造を比較的細かい粒度で表し、コード内の並列性は通常、有向無サイクルグラフ（DAG）としてモデル化される。このようなDAGの各頂点はシーケンシャルコードのセグメントを表し、エッジはそのようなコードセグメント間の優先順位の制約を表す。エッジの末尾の頂点で表されるシーケンシャルコードのセグメントは、エッジの先頭の頂点で表されるシーケンシャルコードのセグメントが実行を開始する前に、実行を完了する必要がある。

並列リアルタイムコードを表現するためのこのようなDAGベースのモデルは、リアルタイムスケジューリング理論のコミュニティで人気があり、これらのモデルを使用してシステムを表現することに基づく多くの重要で興味深い研究が達成されてきた。しかし、様々な理由（その幾つかは列挙され、セクション2で詳細に議論される）により、このようなDAGベースの表現がスケジューラビリティ解析の目的に適さないリアルタイムアプリケーションのクラスが存在し、代替の表現が必要とされている。この論文では、特定の状況下で適切な代替表現の1つを提案する。このモデルでは、コードの内部並列構造を明示的に表現しようとはしない。その代わりに、マルチプロセッサ・プラットフォーム上で並列化可能なコードをスケジューリングしようとするスケジューリング・アルゴリズムにとって最も有用な、並列化可能なコードのいくつかの重要なパラメータを特定し、コードをこれらのパラメータだけで特徴付けられる「ブラックボックス」として見なすことを提案する。さらに、これらのパラメータ値を得るためにコードの内部構造を調べる必要はない。むしろ、これらのパラメータの値は、大規模なシミュレーション実験によって推定することを提案する。パラメータ値の境界を計算できるようにするために、制御された実験室環境でコードを繰り返し実行するのだ。 (このようなアプローチは、確率論的ワーストケース実行時間pWCET解析に関する現在の研究[8]の大規模かつ成長過程に触発されたものである）。測定に基づくアプローチは、通常、絶対確実な正しさが保証されるパラメータ値を提供することができないので、混合クリティカリティスケジューリング文献[9]から、Vestalのアイデア[10]を取り入れ、2つのパラメータセットで1つのタスクを特徴付ける。

##### Organization.

本論文の残りの部分は、以下のように構成されている。セクション2では、現在のモデルが表現するのに適していない並列化可能なリアルタイムコードの関連する特性を特定することで、新しいモデルの動機付けを行い、我々が提案するワークロードとシステムモデルを正式に定義する。セクション3では、提案するモデルの基礎となる先行研究について簡単に述べる。セクション4では、提案モデルを用いて表現されるシステムのスケジューリング アルゴリズムを導出し、その正しさとその他の関連する特性を証明する。我々の全体的な目的は、正しさを保証しつつ、よりリソース効率の良いシステムの実装を得ることである。セクション5では、セクション4で提示したアルゴリズムの効率をさらに向上させるために可能ないくつかの手段を検討する。セクション6では、提案モデルの妥当性、重要性、限界について議論し、今後の研究の方向性を列挙する。
 

## 2 System model: Motivation and Definition

このセクションでは、これまで提案されてきたDAGベースのタスクモデルでは表現しにくい、並列化可能なリアルタイムコードを表現するために提案するモデルの詳細を説明する。まず、モデルの動機付けを非公式に行い、例示を通してモデルの側面を説明する。その後、セクション2.1でモデルの正式な定義を行い、セクション2.2では、このモデルを使って表現されたタスクをスケジューリングするためのアルゴリズムを提案する。

#### Why a new model?
上記セクション1で述べたように、並列リアルタイムコードを表現するためのいくつかの優れたDAGベースのモデルがリアルタイムスケジューリング理論のコミュニティで開発されてきた。しかし、そのようなモデルが適さないことが判明したリアルタイムアプリケーションのクラスがいくつか存在する。これは、以下のような理由によるものである：

1. 並列コードの内部構造は、（条件付きDAGタスクモデル[6,7]などで表現されるような）複数の条件付き依存関係や（境界付き）ループなど、非常に複雑な場合があります。注3：複雑な条件付き並列化可能コードのワーストケース動作を近似する技術は、グローバル固定優先[6]、グローバルEDF[7]、フェデレート[11]などの特定のスケジューリングアルゴリズムに関して提案されている。
2. コードの一部がアプリケーション開発者の組織外から調達されている場合、このコードの提供者は、コードの内部構造を明らかにせず、代わりに実行可能コードのみを提供することによって、知的財産（IP）を保護しようとするかもしれません。(内部構造を特定するために実行コードをリバースエンジニアリングすることは原理的には可能かもしれませんが、そのようなリバースエンジニアリングは面倒でエラーが発生しがちです)。
3. DAGベースのモデルを用いて表現されるシステムの解析のためのアルゴリズムは、DAGのサイズに対して擬似多項式または指数関数的な実行時間を持つ傾向がある。このような実行時間は、従来、アルゴリズムが実際に実用的であるのに十分許容できるほど小さいと考えられてきたが、この状態が将来も続くとは限らない。多くのサイバーフィジカル・リアルタイムシステムでは、納期などの制約は一般的に物理的要因によって決定される。このようなサイバーフィジカル・リアルタイム・システムを実装するプロセッサがますます強力になるにつれて、従来よりもはるかに複雑な処理を組み込むことが可能になり、より大きなDAGとして表現されるようになる。より複雑な処理とそれに伴うより大きなDAGへのこの傾向が続くと、システム設計や解析の際に、これらの大きなDAGのサイズに擬似的に多項式となるランタイムが大きくなりすぎて、実際に使用できなくなる可能性がある。
4. さらに状況を悪化させるのは、並列コードの一部の内部構造をDAG形式で明示的に表現すると、DAGのサイズがコードのサイズの指数関数になる可能性があることです。例えば、マルチプラットフォーム共有メモリ・マルチプロセッシング・プログラミングをサポートするアプリケーション・プログラミング・インターフェース（API）であるOpenMP（[http://www.openmp.org/](http://www.openmp.org/)）で書かれた以下のコードスニペットを考えてみましょう： #pragma omp parallel #pragma omp for (i=0; i<10; i++) { //何かする }.
このコード・スニペットは、$(1+10+1=)$ 12ノードを持つDAGに変換される。しかし、forループの上限の "10 "を "100 "に置き換えると、結果のDAGは102ノードになり、"1000 "に置き換えると1002ノードになる。- プログラムのサイズをASCII文字1文字増やすと、DAGのサイズはほぼ10倍になる。

5. 特に条件付きコードの場合、実行時にコードの真のワーストケース動作が表現されることは非常にまれである。4.条件付きDAGに基づく従来のモデルは、このようなコードを表現するのに適していない可能性がある（ただし、このような条件付きDAGモデルの混合クリティカル[10, 12, 13, 14]拡張は可能性があるが、我々の知る限り、このようなモデルはまだ提案されておらず、研究されていない）。脚注4：例えば、センサーの異常入力を定期的に監視するリアルタイムアプリケーションを考えてみよう。ほとんどの場合、異常な入力は検出されず、計算を実行する必要はあまりありません。しかし、まれに異常入力が検出された場合、そのような入力に対してかなりの追加処理が必要になる。上述したような特徴を1つ以上持つ並列リアルタイム・コードの場合、DAGベースの表現はスケジューラビリティ解析の目的には適さないかもしれない。それでは、そのような表現が提供すべきものについて説明しよう。

#### Identifying relevant characteristics of parallelizable real-time code.
マルチプロセッサ・プラットフォーム上で実行される並列化可能なリアルタイム・コードをモデリングする際の主な目的は、タイミング制約を満たす可能性を高めるために、スケジューリング・アルゴリズムによってコードに存在する可能性のある並列性を利用できるようにすることである。我々がここで興味を持っているのは、予測可能なリアルタイム・システム、つまり実行前にタイミング（およびその他の）正しさを検証できるシステムの開発である。先験的なタイミング検証を可能にするために、並列コンピューティングコミュニティにおける数十年にわたる研究は、並列化可能なコードの以下の2つのタイミングパラメータが特に重要であることを示唆している：

1. **work**パラメータは、全プロセッサで実行されるすべての並列分岐のワーストケースの累積実行時間を示す。非条件付き並列化可能なコードでは、これは単一プロセッサ上でのコードのワーストケース実行時間に等しいことに注意してください（プロセッサの同期による通信オーバーヘッドを無視します）。
2. **span**パラメータは、優先順位に制約のあるコード・シーケンスの最大累積ワース ト・ケース実行時間を示す。これは、利用可能なプロセッサの数に関係なく、コードの実行にかかる時間の下限を示す。計算のスパンは、計算の_クリティカルパスの長さ_とも呼ばれ、累積最悪実行時間がスパンに等しい優先順位制約付きコード片のシーケンスは、計算を通る_クリティカルパス_となる。


これら2つのパラメータの関連性は、優先度に制約されたジョブ(すなわち、DAG)のマルチプロセッサ・スケジューリングが、メイクスパンを最小化することに関するスケジューリング理論のよく知られた結果から生じる。これは、スケジューリング理論で一般的に使用される古典的な3-field $alpha=alpha=mid=beta=mid=gamma$記法における、広く研究されているP|$ prec$|$\mathcal{S}_{max}$問題である[15]。

この問題は長い間、強い意味でNP困難であることが知られている [16]。しかし、Grahamの_list scheduling_ アルゴリズム [17]は、利用可能なジョブが存在する場合、利用可能なプロセッサ上で各瞬間に実行することにより、仕事を保存するスケジュールを構築するものであり、実際にはかなり良好に動作する。リストスケジューリングが以下の保証をすることが示された[17]： $mathcal{S}_{max}$ が特定のDAGを$m$個のプロセッサでスケジューリングできる最小のメー クスパンを示すとすると、このDAGを$m$個のプロセッサでリストスケジューリングして 生成されるスケジュールは、$(2-frac{1}{m})˶timesmathcal{S}_{max}$より大きいメー クスパンを持たない。この結果は、[18]における、このDAGのスケジューリングがメー クスパン$leqfrac{4}{3}}のNP-hardであることを示すhardnessの結 果と合わせて、リストスケジューリングが実際に使用する妥当なアルゴリズム であることを示唆しており、実際、マルチプロセッサ上のDAGのスケジューリン グに使用されるほとんどのランタイムスケジューリングアルゴリズムは、 何らかのリストスケジューリングの変形を使用している。本稿でもそうする。

リストスケジューリングによって生成されるスケジュールのメークスパンの上界は簡単に記述できる。workとspanをスケジューリングされるDAGのworkとspanパラメータとすると、与えられたDAGに対するスケジュールのmakespanが以下の値以下であることが保証されることが[17]で証明されている。

$\frac{\mathrm{work}-\mathrm{span}}{m}+\mathrm{span}$  (1)

DAGのリストスケジューリングによって生成されたスケジュールのメイクスパンの良好な上限は、その作業とスパンのパラメータだけで述べることができます。同様に、DAGが相対デッドラインパラメータ$D$で特徴づけられたリアルタイムのコードを表している場合、$(\frac{\mathrm{work}-\mathrm{span}}{m}+\mathrm{span})\leq D$は、$m$プロセッサのプラットフォーム上でコードがそのデッドラインまでに完了するかどうかを決定するための十分なテストとなります。 したがって、我々は、スケジューラビリティ解析の観点から特に関連性が高いとみなされる、並列リアルタイムコードの作業とスパンのパラメータを特定します

#### A measurement-based approach to parameter estimation.
DAGベースのモデルを使用して表現されるタスクの作業とスパンのパラメータは、DAGの表現に比例して線形時間で計算するのが非常に簡単です（これを行うためのアルゴリズムは[3, 7]で説明されています）。しかし、上で議論したように、私たちの関心は、通常、DAGベースのモデルを使用して便利に表現されない並列コードを特徴づけることにあります。スケジューラビリティ分析のためにこのようなコードの部分を表現する目的で、内部構造を無視し、代わりに作業とスパンのパラメータだけでそれらを特徴づけるよう提案しています。コードの内部構造を知らずに、一般的にコードの作業とスパンのパラメータの正確な値を決定することはできないので、これらのパラメータを推定するために計測ベースのアプローチを使用することをここで提案しています。計測ベースのアプローチは、個々のコードの確率的最悪実行時間（pWCET）[20, 21, 22]の分布を推定するために開発され、Rapita Systemsの[RapiTime](https://www.rapitasystems.com/products/rapitime)のようなpWCETツールで実装されています。ここで、計測ベースのアプローチをどのように適応して並列コードの作業とスパンのパラメータを推定するかについて簡単に説明します。グローバルスケジューリングの実装に関連するオーバーヘッドを無視すると、コードの作業パラメータは、単一のプロセッサ上でその実行を完了するのに必要な時間と等しく、スパンパラメータは、無限数のプロセッサ上でその実行を完了するのに必要な時間と等しいです。したがって、pWCET技術を使用して単一プロセッサ上のWCET分布を推定することで、コードの作業パラメータの確率分布を推定することができます。同様に、以下の方法でスパンパラメータの確率分布を推定することができます。
1. まず、pWCET推定の基盤となる計測ベースの技術を適応して、指定された数のプロセッサ上でのmakespan確率分布を決定します（単一プロセッサ上の完了時間ではなく）。
2. プロセッサの数を繰り返し増やして、並列コードのmakespan分布を推定します。プロセッサの数をさらに増やしても、推定される分布に大きな変化が生じなくなるまで続けます。

#### The proposed model: multiple work and span estimates.
上記で議論したように、pWCET技術の適切な適応と応用により、並列リアルタイムコードの作業とスパンのパラメータの確率分布を推定することが可能です。pWCETコミュニティでは、一般にpWCETベースの技術で絶対的な確実性で正しいと保証される境界を決定することはできないことが理解されています。むしろ、指定された確率的な信頼度/保証レベルで正しいことが保証される境界を提供します（Davis et al. [8]は、このような方法で使用される場合の確率の概念をどのように解釈すべきかについて、考え抜かれた議論を提供しています）。私たちが提案する並列リアルタイムコードを表現するモデルでは、そのようなコードの各部分が_work_、span パラメータ値の_2つ_のペアで特徴づけられることを提案しています。各ペアは作業とスパンの分布の異なる確率しきい値に対応しており、したがって異なる保証レベルで有効です。具体的には、1つの値のペアは非常に保守的であり、非常に高い保証レベルで信頼されるべきであり、もう1つは、まだ比較的安全である一方で、非常に発生しづらいシナリオをカバーしようとはせず、"典型的な"動作をより代表的に示すべきです。私たちは例を通じてこれを示します。

**例 1**: あるコードの特性を以下のように判定できると仮定しましょう。

- その作業パラメータが確率 $p$ で $>120$ であり、確率 $p^{\prime}\ll p$ で $>900$ である。
- そのスパンパラメータが確率 $p$ で $>40$ であり、確率 $p^{\prime}$ で $>600$ である。

このコードを、2つの順序付けられた (作業, スパン) の組み合わせで特徴づけることができます。それは、$(1-p)$ の確率で $(120, 40)$ 以下であることを示し、さらに $(1-p^{\prime})$ の確率で $(900, 600)$ 以下であることを示します。

このコードに関して、より保守的な見積もりの下で**正確性の基準**が満たされる必要があります。たとえば、このコードが相対的な締め切り $D$ 内で実行されることが指定されている場合、作業が $900$ 以下かつスパンが $600$ 以下である限り、Makespan（最大実行時間）が $D$ 以下である必要があります。

#### The proposed run-time scheduling approach.
提案するランタイムスケジューリングアプローチ。正しさは、より保守的なワークとスパンの推定値に関して定義されるので、正しさの要求を満たすためには、より保守的な推定値を仮定してタスクに計算資源を提供しなければならない。しかし、タスクの実行時動作は、より保守的でないパラメータ推定値によって制限される可能性が非常に高いため、より保守的な仮定の下で静的に適切なリソースを提供することは、実行時に計算リソースを大幅に浪費する可能性が高い。そのような浪費を改善する1つの方法は、プロビジョニングされたリソースの一部を「リザーブ」にしておくことです。おそらく、いくつかのプロセッサをスリープモードにするか、バックグラウンド（非リアルタイム）作業を実行させます。次の例はそれを示している。

**例 2**: 例 1のコードが、相対的な締め切りが 690 である 10 プロセッサのプラットフォームでスケジュールされると仮定します。式 1に従うと、10 プロセッサ上でのスケジュールのMakespan（最大実行時間）は以下のように計算されます。

$\frac{900-600}{10}+600=(30+600)=\mathbf{630}$

より保守的な作業およびスパンの見積もりが成立すると仮定すると、したがって、正確性が保証されます。

この計算により、10プロセッサのプラットフォームでのMakespanが最大で630であるため、相対的な締め切り690を満たすことが確認されます。

実際には、正確性を保証するためには10プロセッサは必要ありません。このコードを4つのプロセッサで実行する場合、式1に従って得られるMakespanの制約は次のように計算できます：

$\left(\frac{900-600}{4}\right)+600=\mathbf{675}$

したがって、提供された10プロセッサのうち6つを単にオフにして、残りの4つでタスクを実行することで、ランタイムアルゴリズムはエネルギーの節約を実現できるでしょう。

7つのプロセッサをオフにできるでしょうか？3つのプロセッサで実行した場合、Expression 1によって計算されるMakespanの制約は次のようになります：

$\left(\frac{900-600}{3}\right)+600=\mathbf{700}$

700は指定された相対的な締め切りである690を超えるため、7つのプロセッサをオフにした場合、システムがそのより保守的なパラメータではなく、より保守的なパラメータによって予想されるよりも悪く振る舞う可能性がある場合、締め切りを逃す可能性があることが確認されます。

したがって、正確性を確保するためには、スリープモードにならないように少なくとも4つのプロセッサが必要であると結論できます。

この論文で説明されているランタイムアルゴリズムは、より保守的なパラメータが「ほとんどの場合」成立すると非常に確率が高いシステム向けに設計されています。つまり、$p$ の値自体が非常に小さいと仮定されています。この例においても、この前提が成り立つ場合を考えてみましょう。

* ランタイムスケジューリングアルゴリズムは、最初にシステムを3つのプロセッサでスケジュールし、残りの7つのプロセッサをスリープモードにします。
* 実行がある事前に計算された時間に完了していない場合、スリープモードのプロセッサを起動し、このタスクを実行するために10つのプロセッサをすべて使用可能にします。

このアルゴリズムによれば、より保守的なタスクパラメータが成立する場合、タスクは指定された締め切りである690までに完了し、**正確性**が確立されます。さらに、事前計算された時間に完了しない可能性が $\leq p$ であり、したがって残りの7つのプロセッサを起動する必要がある可能性があります。
このアルゴリズムにより、より保守的なパラメータがほとんどの場合成立するシステムに対して、正確性を確保しながらエネルギーを節約することができます。

**効率性**を評価するために、この例において $p=0.05$ と仮定しましょう。これは、より保守的なパラメータが95%の確率で成立することを示しています。したがって、指定された時間にタスクが完了しない確率は $\leq 5\%$ であり、必要なプロセッサの期待値は次のように計算できます：

$\left(0.95\times 3+0.05\times 10\right)=\left(2.85+0.5\right)=\mathbf{3.35}$

この場合、我々のランタイムスケジューリングアルゴリズムを使用しない場合に必要とされる4つのプロセッサと比べて、必要なプロセッサ数は3.35となります。

このアルゴリズムを使用することで、より保守的なパラメータがほとんどの場合成立する場合に、タスクを効率的に実行するために必要なプロセッサ数を減らすことができ、効率性を向上させることができます。

上記の例1と例2は、この論文で提案されているタスクモデルと、そのモデルに基づいてスケジュールされるタスクのランタイム戦略を示しています。次に、我々はセクション2.1でタスクモデルを正式に定義し、セクション2.2でランタイムスケジューラを定義します。

### 2.1 System Model
私たちは、仮定するワークロードモデルの詳細を記述することで、モデルの正式な定義を提供します。モデル化を目指すワークロードは、次のパラメータのリストで特徴づけられる、単一の並列化可能なリアルタイムコードから構成されています：

$<work_O, span_O, work_N, span_N, D>$

これらのパラメータは次のように解釈されます：

1. $(\operatorname{\mathrm{work}}{O},\operatorname{\mathrm{span}}{O})$：これらは作業とスパンのパラメータの真の「最悪ケース」の値の非常に保守的な推定値を表しています。上記で議論したように、これらの推定値は、確率的最悪実行時間（pWCET）の分布を推定するために開発された計測ベースの技術を使用して得られることが期待されます。

2. $D$は_relative deadline_ パラメータを示します：正しい実行のために、ジョブのmakespanは$D$を超えてはならないと要求されます。ここで強調したいのは、タイミングの正確さは、$(\operatorname{\mathrm{work}}{O},\operatorname{\mathrm{span}}{O})$パラメータの推定値が正しいと仮定して指定されるということです：コードは、作業とスパンのパラメータがそれぞれ$\operatorname{\mathrm{work}}{O}$および$\operatorname{\mathrm{span}}{O}$よりも大きくない限り、指定された相対締め切り$D$内で実行を完了する必要があります。

3. $(\operatorname{\mathrm{work}}{N},\operatorname{\mathrm{span}}{N})$：これらは、作業とスパンのパラメータの値の保守的でない推定値です。作業とスパンの実際の値がそれぞれ$\operatorname{\mathrm{work}}{N}$および$\operatorname{\mathrm{span}}{N}$よりも_非常に_確実に大きくないと期待されます。$\operatorname{\mathrm{work}}{N}$および$\operatorname{\mathrm{span}}{N}$のサブスクリプト「$N$」は「標準」[23]を意味します。) これらのパラメータ推定は、正確さを定義するための役割は果たさず、むしろ（例1および2で見たように）、よりリソース効率の良いスケジューリング戦略を考案するためにその値が使用されることがあります。したがって、$\operatorname{\mathrm{work}}{O}$および$\operatorname{\mathrm{span}}{O}$のパラメータと同様に、その値が正しく割り当てられることはそれほど重要ではありません：$\operatorname{\mathrm{work}}{O}$および$\operatorname{\mathrm{span}}{O}$の誤って推定された値は、期限を逃す可能性があるという意味でタイミングの正確さを損なう可能性がありますが、$\operatorname{\mathrm{work}}{N}$および$\operatorname{\mathrm{span}}{N}$の誤って推定された値は単に効率の低い実装になるだけです。$\operatorname{\mathrm{work}}{N}\leq\operatorname{\mathrm{work}}{O}$および$\operatorname{\mathrm{span}}{N}\leq\operatorname{\mathrm{span}}{O}$であると仮定します。（これらの仮定が成立しない状況にも結果は容易に拡張されますが、$(\operatorname{\mathrm{work}}{N},\operatorname{\mathrm{span}}{N})$パラメータが$(\operatorname{\mathrm{work}}{O},\operatorname{\mathrm{span}}{O})$パラメータよりも保守的でない推定を示す定義そのもののため、これらの仮定を緩和する根拠は見当たりません。）

この論文では、同一のプロセッサバンクに対する単一のタスクのスケジューリングに焦点を当てています。また、私たちの結果は、マルチプロセッサのスケジューリングの「連邦パラダイム」[24]の下での定期的またはスポラディックなリアルタイムDAG（有向非循環グラフ）のスケジューリングにも直接適用できます。前述の条件を満たす限り、各定期的/スポラディックなタスクがその相対的な締め切りパラメータがその周期パラメータよりも大きくない制約つき締め切りタスクである場合です。我々は、我々のアプローチが定期的なタスクのスケジューリングシステムに特に適していると考えています。このようなタスクの場合、$(\operatorname{\mathrm{work}}_{N},\operatorname{\mathrm{span}}_{N})$ パラメータが、そのタスクのほとんどの呼び出し（"dag-jobs" [3]とも呼ばれる）の動作を制約すると予想しており、まれな場合にはこれらの制約を超えることがあります。したがって、割り当てられた多くのプロセッサは、ほとんどの時間スリープモードにとどまり、まれにスリープモードのプロセッサを起動する必要があるまで、スリープモードにとどまるでしょう。その後、そのdag-jobの実行が完了したら、プロセッサを再びスリープモードに戻すことができます。

### 2.2 The Scheduling Algorithm
上記で指定されたタスク：

$<\operatorname{work}{O}, \operatorname{span}{O}, \operatorname{work}{N}, \operatorname{span}{N}, D >$

このタスクを $m$ 個の同一のプロセッサからなるプラットフォームで実行する場合、まず実行前のスケジュラビリティ分析を行います。この分析により、タスクを $m$ 個のプロセッサで正確性を保証する方法でスケジュールできるかどうかが決定されます。ここで、正確性は、タスクがその作業パラメータが $\leq \operatorname{work}{O}$ であり、スパンパラメータが $\leq \operatorname{span}{O}$ の場合に締め切りを満たすことを要求するものです。Expression 1によれば、これは次の条件を満たす場合に保証されます：

$(\frac{\operatorname{work}_{O}-\operatorname{span}_{O}}{m}+\operatorname {span}_{O})\leq D;$ (2)

この条件が成立する場合、タスクは指定された相対締め切り $D$ 内で正確に実行されることが保証されます。

条件2が成立しない場合、スケジューリングアルゴリズムは失敗と宣言します。これは、このインスタンスをタイミングの正確性を保証する方法でスケジュールできないことを意味します。それ以外の場合、アルゴリズムは $m_{N}$ および $\mathcal{S}{N}$ という2つの値を計算します。これらの値の計算方法はセクション4.2で導出されます。計算されたこれらのパラメータは、次のように解釈されます。より保守的でない作業とスパンパラメータの推定が正確である場合（つまり、タスクの作業が $\leq \operatorname{work}{N}$ であり、スパンが $\leq \operatorname{span}{N}$ である場合）、リストスケジューリングは、タスクを $m{N}$ のプロセッサ上で $\mathcal{S}_{N}$ を超えないMakespanでスケジュールできることを意味します。

#### Run-time scheduling.
このタスクで表される並列化可能なリアルタイムコードが、実行時のある時間インスタント $t_{o}$ にアクティブ化されたと仮定します。

1. スケジューラはタイマーを設定し、時間インスタント $(t_{o}+\mathcal{S}_{N})$ でタイマーが作動するようにし、タスクを $m_{N}$ 個のプロセッサ上でリストスケジューリングアルゴリズム [17] を使用して実行を開始します。このタスクに割り当てられた残りの $(m-m_{N})$ 個のプロセッサはスリープモードに配置またはスリープモードに留まります。
2. タスクが時間インスタント $(t_{o}+\mathcal{S}_{N})$ までに実行が完了しない場合、スケジューラは $(m-m_{N})$ 個のスリーププロセッサを起動し、タスクの残りの部分を全ての $m$ 個のプロセッサのバンクでリストスケジューリングを使用して実行します。
3. セクション2.1で述べたように、定期的なタスクの場合、これらの起動されたプロセッサは現在のdag-jobの実行が完了すると、スリープモードに戻されます。

この方法により、タスクの効率的なスケジューリングが実現され、少ないプロセッサで効率的に実行できる場合にはスリープモードに保持され、必要に応じて起動されます。

セクション4では、正確性を保証するために $m_{N}$ および $\mathcal{S}_{N}$ の値がどのように計算されるかについて説明します。このアルゴリズムにより、タスクの作業パラメータが $\leq\operatorname{work}_{O}$ であり、スパンパラメータが $\leq\operatorname{span}_{O}$ の場合、タスクは到着から $D$ 時間単位以内に実行が完了します。

このセクションを閉じる前に、私たちの実行時スケジューラの動作を説明する例を示します。

**例3** 例1および例2で議論されたインスタンスを考えてみましょう。セクション2の表記法では、このタスクは次のパラメータで表されます：

$<\operatorname{work}_{O}, \operatorname{span}_{O}, \operatorname{work}_{N}, \operatorname{span}_{N}, D> = <900, 600, 120, 40, 690>$

このタスクは $m=10$ 個のプロセッサでスケジュールされます。Example 4では、セクション4のアルゴリズムがパラメータ $m_{N}$ に値 $3$ を割り当て、パラメータ $\mathcal{S}_{N}$ に値 $66\frac{2}{3}$ を割り当てることを示します。したがって、私たちの実行時スケジューラはこのタスクを最初に $3$ 個のプロセッサでスケジュールします。タスクがその workN, spanN パラメータによって指定されたように振る舞う場合、Expression 1によれば、Makespan は以下のようになります：

$
\left(\frac{120 - 40}{3} + 40\right) = 26\frac{2}{3} + 40 = 66\frac{2}{3}
$

したがって、追加の7つのプロセッサは必要ありません。タスクが $66\frac{2}{3}$ の時間インスタントまでに完了しない場合、すべての10個のプロセッサがこのタスクの実行に利用可能となり、セクション4の結果により、タスクは指定された締め切り時間である時間インスタント690までに正しく実行されることが確認できます。

## 3 Related Work
ここで提案している、並列化可能なタスクのモデリングと実行時スケジューリングのアプローチは、並列計算、混合臨界度スケジューリング、および確率的WCETの研究からのインスピレーションを得ています。

上記のセクション2で述べたように、makespanを最小化するためのDAGのスケジューリング問題（3フィールド表記[15]のP| prec| Smax問題）は、「伝統的な」スケジューリング理論で非常に広く研究されてきました。この問題の固有の計算困難さ[16]と良好な近似の存在（リストスケジューリング[17]とその関連するmakespan上限 – 不等式1によって代表される）を考えると、並列計算コミュニティは、並列化可能な計算ワークロードの妥当な代理として作業とスパンのパラメータに焦点を当てるようになりました。これは私たちが提案しているアプローチの基礎となる基本的なアイデアの1つです。

異なる確証レベルで信頼できると考えられる複数の値をタスクのパラメータに指定するという概念は、Vestal[10]によって提案され、混合臨界度スケジューリング理論の基礎となっています。Vestalモデルを探求する研究が多数存在し、[14]で調査結果を参照できます。混合臨界度の並列タスクのスケジューリングを研究する中で、Liら[23]は、各タスクが低臨界度と高臨界度で異なる作業とスパンのパラメータで特徴付けられるモデルを初めて提案しました。こちらが今回詳しく研究しているモデルです。（[23]の研究の全体的な文脈は私たちのものとは大きく異なります。

私たちは、[23]の用語で言えば、名目ケースでのプロセッサの使用数を最小化しつつ、過負荷のケースで期限を確実に遵守することを目的として、単一の並列化可能なリアルタイムタスクのスケジューリングを検討していますが、[23]は良好な容量増加の境界を持つ混合臨界度スケジューリングアルゴリズムの考案を目的としています）。また、確率的な最悪実行時間分布（pWCET）の推定のための計測ベースの技術に関する既存の大量の研究（例：[20, 21, 22]）からのアイディアも取り入れています。私たちのスケジューリングフレームワークの正確さは、pWCET推定技術の有効性と精度に非常に強く依存しており、実質的には、より保守的な推定であるworkOとspanOが正確な上限であるという仮定の下で、正確なタイミングの振る舞い（期限を守る）を保証しています。

対照的に、workNおよびspanNの誤った推定は正確さを損なうことはありませんが、効率に悪影響を及ぼす可能性があります。これらのパラメータは、最悪ケースのパラメータ値の推定として考えられるよりも、Chisholmら[25]が供給されたパラメータ値と呼んでいるもの、またはLiら[23]が名目のパラメータ値と呼んでいるものに近いかもしれません。これは典型的な、または一般的な動作を表す値であり、たとえば、平均ケースのパラメータ値をやや膨らませることで取得できるかもしれません。

## 4 Scheduling Algorithm Derivation and Analysis
セクション2.1で説明されたように、以下のパラメータで特徴付けられるタスク

$\text{work}_{O},\text{span}_{O},\text{work}_{N},\text{span}_{N},D$

と、そのタスクを実行するための$m$個のプロセッサについて、このセクションでは、上記のセクション2.2で述べた実行時のスケジューリングアルゴリズムが、タスクの到着から$D$時間単位以内にそのタスクの実行を完了させるために、$m_{N}$と$\mathcal{S}_{N}$の値をどのように計算するかについて議論します。セクション4.1では、$m_{N}$と$\mathcal{S}_{N}$の値が既に知られていると仮定して始め、これらの$m_{N}$と$\mathcal{S}_{N}$の値が与えられた場合のタイミングの正確さを保証するための十分条件を導き出します。その後、セクション4.2で、これらの十分条件が満たされるように、$m_{N}$と$\mathcal{S}_{N}$にどのように値を割り当てるかについて説明します。

### 4.1 Sufficient Schedulability Conditions

$m_{N}$ と $\mathcal{S}_{N}$ の値が与えられた場合（$0<m_{N}\leq m$ および $0\leq\mathcal{S}_{N}\leq D$）、実行時アルゴリズムはリストスケジューリングを使用してタスクを $m_{N}$ 個のプロセッサでスケジュールします。タスクが $\mathcal{S}_{N}$ 時間単位以内に実行を完了する場合、$\mathcal{S}_{N}\leq D$ であるため、正確性は維持されます。タスクが $\mathcal{S}_{N}$ の時間インスタントまでに完了しない場合の正確性を確保するための十分な条件を決定する残りの部分をこのセクションで行います。

図1: 並列タスクは時間インスタント0で実行を開始し、時間インスタントDで締め切りがあります。それは $[0,\mathcal{S}_{N})$ の間で $m_{N}$ 個のプロセッサ上で実行され、 $[\mathcal{S}_{N},D)$ の間で $m$ 個のプロセッサ上で実行されます。 （したがって、x軸は時間を示し、y軸はプロセッサを示します。）

# ここに図を入れる

図1は、タスクが $\mathcal{S}_{N}$ の時間単位内に実行を完了しない場合に利用可能なプロセッサを示しており、これにより実行時スケジューラが $[0,\mathcal{S}_{N})$ の間スリープモードにあった $(m-m_{N})$ 個のプロセッサを起動します。これらの利用可能なプロセッサ上で実行される場合、その作業パラメータが $\text{work}_{O}$ と同じぐらい大きいかもしれないし、スパンパラメータが $\text{span}_{O}$ と同じぐらい大きいかもしれないと仮定して、タスクが時間インスタント $D$ で締め切りを満たすための条件を導出します。

$\text{work}^{\prime}$ および $\text{span}^{\prime}$ を、時間インスタント $\mathcal{S}_{N}$ での並列タスクの計算量の作業パラメータとスパンパラメータとして定義しましょう（これらは $>0$ です、なぜならタスクは時間インスタント $\mathcal{S}_{N}$ までに実行が完了しないと仮定しているからです）。この残りの計算は $m$ 個のプロセッサで実行されます。式1によれば、全体のメイクスパンは次のように上から制約されます。

$\mathcal{S}_{N}+\left(\frac{\text{work}^{\prime}-\text{span}^{\prime}}{m}+ \text{span}^{\prime}\right)$ (3)

時間インスタント $\mathcal{S}_{N}$ での残りのスパンは $\text{span}^{\prime}$ であり、タスクのクリティカルパスの $(\text{span}_{O}-\text{span}^{\prime})$ が $[0,\mathcal{S}_{N}]$ の間に実行されています。クリティカルパスが実行されていない各瞬間では、すべての $m_{N}$ 個のプロセッサがクリティカルパスに関係のないタスクを実行している必要があります。したがって、$[0,\mathcal{S}_{N})$ の間に発生する実行の合計は少なくとも次のようになります。

$(\mathcal{S}_{N}-(\text{span}_{O}-\text{span}^{\prime}))\times m_{N}+(\text{span}_{O}-\text{span}^{\prime})$ 

これから次のことがわかります。

$$
\begin{align*}
\text{work}^{\prime} & \leq \text{work}_{O}-\mathcal{S}_{N}\times m_{N}+(\text{span}_{O}- \text{span}^{\prime})\times m_{N}-(\text{span}_{O}-\text{span}^{\prime}) \\
& = \text{work}_{O}-\mathcal{S}_{N}\times m_{N}+(\text{span}_{O}- \text{span}^{\prime})\times(m_{N}-1) \\
& = \text{work}_{O}-\mathcal{S}_{N}\times m_{N}+\text{span}_{O}\times (m_{N}-1)-\text{span}^{\prime}\times(m_{N}-1) (4)
\end{align*}
$$

不等式4を式3に代入すると、全体のメイクスパンに対する次の上限値が得られます：

$$
\begin{align*}
&\mathcal{S}_{N}+\left(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)-\text{span}^{\prime}\times(m_{N}-1 )-\text{span}^{\prime}}{m}+\text{span}^{\prime}\right) \\
&= \mathcal{S}_{N}+\left(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)-\text{span}^{\prime}\times m_{N}}{m}+\text{span}^{\prime}\right) \\
&= \mathcal{S}_{N}+\left(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)}{m}-\text{span}^{\prime}\times \frac{m_{N}}{m}+\text{span}^{\prime}\right) \\
&= \mathcal{S}_{N}+\left(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)}{m}+\text{span}^{\prime}\times (1-\frac{m_{N}}{m})\right)
\end{align*} (5)
$$

$m_{N}\leq m$ であるため、式5は $\text{span}^{\prime}$ ができるだけ大きい場合に最大化されます。つまり、$\text{span}^{\prime}=\text{span}_{O}$ です（物理的な解釈は、クリティカルパス上のジョブが時間インスタント $\mathcal{S}_{N}$ の前に実行されない場合、最悪の場合が発生する：代わりにクリティカルパス全体が $\mathcal{S}_{N}$ の後に実行されます）。式5に $\text{span}^{\prime}\leftarrow\text{span}_{O}$ を代入すると、次の全体のメイクスパンの上限値が得られます：

$$
\begin{align*}
&\mathcal{S}_{N}+\left(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)}{m}+\text{span}_{O}\times(1- \frac{m_{N}}{m})\right) \\
&= \mathcal{S}_{N}+\left(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}-\text{span}_{O}}{m}+\text{span}_{O}\right)
\end{align*}
$$

この上限値が $D$ 以下であることにより、正確性が保証されます：

$$
\begin{align*}
&\left(\mathcal{S}_{N}+\left(\frac{\text{work}_{O}-\mathcal{S}_{N }\times m_{N}-\text{span}_{O}}{m}+\text{span}_{O}\right)\right) \leq D \\
&\Leftrightarrow \left(\mathcal{S}_{N}-\frac{\mathcal{S}_{N}\times m_{N}}{m}\right) \leq \left(D-\frac{\text{work}_{O}-\text{span}_{O}}{m}-\text{span}_{O}\right) \\
&\Leftrightarrow \mathcal{S}_{N}\left(1-\frac{m_{N}}{m}\right) \leq \left(D-\frac{\text{work}_{O}-\text{span}_{O}}{m}-\text{span}_{O}\right)
\end{align*}
$$

上記の式6は、タイミングの正確性を保証するための十分なスケジュラビリティ条件であり、式6を満たす $m_{N}$ と $\mathcal{S}_{N}$ の値は、タイミングの正確性を保証します。

### 4.2 Computing $m_{N}$ and $\mathcal{S}_{N}$
前述のセクション4.1で、正確性を保証するために、私たちのスケジューリングアルゴリズムは上記の条件6を満たすように、パラメータ $m_{N}$ と $\mathcal{S}_{N}$ を選択する必要があることを見ました。さらに、追加の目標として、_効率性_ があります：$m_{N}$ の値が小さいほど良いです。なぜなら、残りの $(m-m_{N})$ プロセッサをスリープモードにすることができるからです。このセクションでは、アルゴリズムがどのようにしてそのような値を計算するかを説明します。

$m_{N}$ パラメータに値を割り当てるための合理的なアプローチの一つは、タスクのワークとスパンのパラメータ、$\text{work}_{N}$ と $\text{span}_{N}$ を使用することです。これらのパラメータがタスクの「典型的な」呼び出しのワークとスパンの値を制約すると仮定すると、不等式1により、$m_{N}$ プロセッサ上の典型的な呼び出しは、$((\text{work}_{N}-\text{span}_{N})/m_{N}+\text{span}_{N})$ 以上のメイクスパンを持つことはありません。したがって、$\mathcal{S}_{N}$ に次のように値を割り当てることができます：

$\mathcal{S}_{N}\leftarrow(\frac{\text{work}_{N}-\text{span}_{N}}{m_{N}} +\text{span}_{N})$ (7)

この値を$\mathcal{S}_{N}$の式6に代入すると、次のようになります。

$(\frac{\text{work}_{N}-\text{span}_{N}}{m_{N}}+\text{span}_{N}) \times(1-\frac{m_{N}}{m})\leq(D-\frac{\text{work}_{O}-\text{ span}_{O}}{m}-\text{span}_{O})$ (8)

これは十分なスケジューラビリティ条件です。この式では、$m_{N}$以外の各項が定数であるため、この式を$m_{N}$の2次式の形に代数的に簡略化できます。この2次式を解き、天井関数を取ることで（プロセッサの数$m_{N}$は整数でなければならないため）、必要な値を得ることができます。一旦$m_{N}$が計算されたら、$\mathcal{S}_{N}$の値は式7から得ることができます。以下に例を示します。$m_{N}$および$\mathcal{S}_{N}$を計算するアルゴリズムは、この例の後に疑似コード形式で提供されています。

**例4** 再度、Examples 1および2で議論されたインスタンスを考えてみましょう：

$<\text{work}_{O},\text{span}_{O},\text{work}_{N},\text{ span}_{N},D> = <900,600,120,40,690>$

これを$m=10$のプロセッサでスケジュールすると仮定します。

これらの値を式8に代入すると、以下のようになります：

$$(\frac{work_{N}-\text{span}_{N}}{m_{N}}+\text{span}_{N}) \times(1-\frac{m_{N}}{m})\leq(D-\frac{\text{work}_{O}-\text{ span}_{O}}{m}-\text{span}_{O})$$

$$\equiv (\frac{120-40}{m_{N}}+40)\times(1-\frac{m_{N}}{ 10})\leq(690-\frac{900-600}{10}-600)$$

$$\equiv (\frac{80}{m_{N}}+40)\times(1-\frac{m_{N}}{10} )\leq 60$$

$$\equiv 40\cdot(\frac{2}{m_{N}}+1)\times(1-\frac{m_{N} }{10})\leq 60$$

$$\equiv 2\cdot(\frac{2+m_{N}}{m_{N}})\times(\frac{10-m_ {N}}{10})\leq 3$$

$$\equiv (2+m_{N})\times(10-m_{N})\leq 15m_{N}$$

$$\equiv 20+8m_{N}-m_{N}^{2}\leq 15m_{N}$$

$$\equiv m_{N}^{2}+7m_{N}-20\geq 0$$

これから、以下の結論を得ます：
$$m_{N}\geq\frac{-7+\sqrt{129}}{2}\ \approx\ 2.18$$


プロセッサ数は整数である必要があるため、$m_{N}\gets 3$と結論します。対応する$\mathcal{S}_{N}$の値は次のようになります：

$(\frac{\text{work}_{N}-\text{span}_{N}}{m_{N}}+\text{span}_{N})= (\frac{120-40}{3}+40)=26\frac{2}{3}+40=66\frac{2}{3}$


### Algorithm 1 Computing values for $m_{N},\mathcal{S}_{N}$.
----
Input: $(\langle\text{work}_{O},\text{span}_{O},\text{work}_{N},\text{span}_{N},D \rangle, m)$ 
Output: failure, or values for $m_{N}$, $\mathcal{S}_{N}$

1. begin
2.   if $(m < \lceil(\text{work}_{O}-\text{span}_{O})/(D-\text{span}_{O})\rceil)$ then
3.   return $(\text{failure})$  /* The test of Inequality 1 cannot guarantee that the deadline will be met on $m$ processors */
4.   end if
5.   $A \leftarrow \text{span}_{N}$
6.   $B \gets m \times (D - (\text{span}_{O} + \text{span}_{N})) - (\text{work}_{O} - \text{span}_{O}) + (\text{work}_{N} - \text{span}_{N})$
7.   $C \leftarrow (-1) \times m \times (\text{work}_{N} - \text{span}_{N})$
8.   $m_{N} \leftarrow \lceil(-1 \times B + (\sqrt{B^{2} - 4 \times A \times C}) / (2 \times A))\rceil$
9.   $\mathcal{S}_{N} \leftarrow \text{span}_{N} + (\text{work}_{N} - \text{span}_{N}) / m_{N}$
10.  return $(m_{N}, \mathcal{S}_{N})$
11. end while
----


#### Pseudo-code representation
式8を以下の形式に書き換えることができることを確認できます：

$$A\times m_{N}^{2}+B\times m_{N}+C\geq 0$$

ここで、$A,B,C$は次の値が割り当てられています：

$$A \leftarrow \text{span}_{N}$$
$$B \leftarrow (m(D-(\text{span}_{O}+\text{span}_{N}))-(\text {work}_{O}-\text{span}_{O})+(\text{work}_{N}-\text{span}_{N}))$$
$$C \leftarrow -1\times m\times(\text{work}_{N}-\text{span}_{N})$$

アルゴリズムリスト1の擬似コードは、この二次不等式の正の根を見つけます。天井関数を使って必要なプロセッサの数$m_{N}$を示し、この計算はLine 8で行われます。Line 9では、計算された$m_{N}$の値を使用して$\mathcal{S}_{N}$に割り当てる値を決定します。

#### Run-time complexity
Algorithm 1はループや再帰呼び出しがない直線的なコードから成り立っています。したがって、タスクを指定するパラメータを入力として与えられた場合、Algorithm 1の実行時間は定数であることが明らかであり、$\Theta(1)$です。


## 5 Achieving Greater Efficiency: A More Aggressive Approach
効率性（「通常の場合」に使用されるプロセッサの数を減少させる）を追求しつつ、正確性を維持する（タスクの振る舞いが$\text{work}_{O}$および$\text{span}_{O}$の最悪ケースの境界を超えない限り、期限を必ず守ることを保証する）ために、上記のセクション4.2で導出されたアプローチは、ノミナルのパラメータ値$\text{work}_{N}$および$\text{span}_{N}$を使用して$m_{N}$および$\mathcal{S}_{N}$に値を割り当てます。このセクションでは、あらゆる方法で正確性を妥協することなく、おそらく更なる効率性を達成するための、より積極的なアプローチを提案します。このより積極的なアプローチは、次の点に関する洞察を利用しています：(i) システムの実行時の振る舞いの確率的な特性化；および (ii) リストスケジューリングの典型的な振る舞い

#### The probabilistic characterization of run-time behavior.
セクション2で議論されたように（例1および2で示されているように）、$\text{work}_{N}$および$\text{span}_{N}$のパラメータ値が正しい確率が与えられる場合があります。例えば、実際の作業が$\text{work}_{N}$を超える確率が$\leq 0.05$、実際のスパンが$\text{span}_{N}$を超える確率が$\leq 0.05$であると主張できるかもしれません。このしきい値の確率0.05は、$(m-m_{N})$のスリープ中のプロセッサが起動する必要がある確率が$\leq 0.05$であることを望むために選ばれたかもしれません。そうであるならば、セクション4.2で述べられている$\mathcal{S}_{N}$と$m_{N}$の計算方法は、作業とスパンの分布が完全に相関していない可能性があるため、過度に保守的であるかもしれません - もしそうでないなら、作業が$\text{work}_{N}$を超え、スパンが$\text{span}_{N}$を超える確率は、タスクの特定の実行中に0.05よりも小さいです。（極端な場合、2つの分布がほぼ独立している場合、確率は$0.05^{2}$、すなわち0.0025に近く、これは求められているしきい値の確率0.05よりもはるかに小さい値です。）

#### Some observations on List Scheduling.
セクション4.2で、実際の作業とスパンのパラメータがそれぞれ$\text{work}_{N}$と$\text{span}_{N}$を超えないと仮定して、Expression 1を使用して$m_{N}$と$\mathcal{S}_{N}$の値を割り当てました。これは、$m_{N}$プロセッサ上で$\mathcal{S}_{N}$の期間内に計算が実行を完了することを保証する方法でした。Expression 1は、DAGのList Scheduling生成スケジュールのmakespanの上限であり、この上限はDAGが非常に特定の構造形式を持っている場合、および/またはList Schedulingが特定のスケジューリング決定のシーケンスを行う場合にのみ厳密です（そして、DAGの各ノードがその全体のWCETで実行される場合のみ）。ランダムに生成されたグラフを使用したシミュレーション実験は、これらの構造とスケジューリング決定が比較的まれであることを示しているようです。ランダムに生成されたグラフの場合、実際のリストスケジューリング生成スケジュールのmakespanは、Expression 1の上限と次の明らかな下限の間のインターバルの下端に近づく傾向があります。

$\max(\frac{\text{work}}{m},\text{span})$ (9)
本セクションでは、$\mathcal{S}_{N}$ と $m_{N}$ の値を過度に楽観的に設定することが正確性に影響を与えることはないことを強調します。その唯一の影響は、効率に関連しており、保持可能なプロセッサの数や、システムの実行中にこれらのプロセッサをオンにする必要がある可能性に関連しています。したがって、ランタイム中のリソース効率を向上させるための、より積極的なアプローチの1つは、Expression 7に従うのではなく、Expressions 9と1の下限と上限の間の値を$\mathcal{S}_{N}$に割り当てることです。

$\mathcal{S}_{N}\leftarrow\max(\frac{\text{work}_{N}}{m_{N}},\text{span }_{N})+\alpha\cdot[(\frac{\text{work}_{N}-\text{span}_{N }}{m}+\text{span}_{N})-\max(\frac{\text{work}_{N}}{m_{N}},\text{ span}_{N})] $ (10)

ここで、$\alpha$は0から1までの"チューニング"パラメータです。$\alpha$の値が小さいほど、$\mathcal{S}_{N}$の選択はより積極的です。（ここでのトレードオフの直感的な解釈は、$\alpha$の値が小さいほど、オフにできるプロセッサの数は増えますが、タスクの実行中にそれらを起動する必要がある可能性も高くなります。）Table 1のランダムに生成されたDAGに対して、$\alpha\geq 0.208$ の値が安全であったでしょう：ランタイム中、タスクの実行時の動作がその公称パラメータ$\text{work}_{N}$と$\text{span}_{N}$を侵害しない限り、スリープ中のプロセッサは起動されません。

Expression 10を使用して$\mathcal{S}_{N}$の値を割り当てる場合、Expression 7の代わりにそれを使用すると、Expression 8を解決することで$m_{N}$の所望の値が得られるとは限らなくなります。Expression 7の代わりにExpression 10を使用する場合の$m_{N}$の閉形式の解を求める試みはしていません。むしろ、$[1,m)$の範囲で$m_{N}$の候補値を繰り返し処理し、この$m_{N}$の値と、Expression 10に従って計算される$\mathcal{S}_{N}$の結果の値が、Condition 6を真と評価する最初の値で停止します。したがって、このより積極的なアプローチにより、$m_{N}$と$\mathcal{S}_{N}$を計算するランタイムの複雑さは$\Theta(m)$となります。ここで、$m$は利用可能なプロセッサの数を示します。バイナリサーチの考え方を直接適用することで、これを$\Theta(\log m)$に削減することができます。

この新しい方法は、所望の条件を満たすまで可能な$m_{N}$の値を順次探索するため、最悪の場合には全プロセッサ数$m$に応じてその時間がかかります。しかし、バイナリサーチを使用することで、より迅速に所望の$m_{N}$の値を見つけることができ、計算時間を大幅に短縮することができます。

----
表1: 10プロセッサプラットフォーム上のランダムに生成されたDAGの実際のmakespan、下限、および上限。各グラフは1000の頂点を持ち、各行は最初の列で指定された辺の数を持つ1000頂点のDAGに対応しています。最後の列は、(実際のmakespan - 下限) ÷ (上限 - 下限)の比率を示しています。小さい値は、実際のmakespanが下限に近いことを示しています。（このテーブルを生成するための実験は、セクション5.1で詳しく説明されています。）

| **# edges** | **Lower (Exp. 9)** | **Actual** | **Upper (Exp. 1)** | **Ratio** |
|-------------|--------------------|------------|--------------------|-----------|
| 977        | 2627               | 2667      | 2818               | 0.208     |
| 2017       | 2539               | 2587      | 2889               | 0.137     |
| 4921       | 2567               | 2603      | 3222               | 0.055     |
| 9935       | 2554               | 2709      | 3725               | 0.132     |
| 20094      | 2599               | 2977      | 4774               | 0.174     |
| 39935      | 4056               | 4113      | 6154               | 0.027     |
| 50036      | 4454               | 4480      | 6491               | 0.013     |
| 60212      | 5674               | 5674      | 7658               | 0.000     |
----


### 5.1 The Experiments Reported in Table 1
Table 1にデータを入力するための実験手順を簡単に説明します。グラフは、ランダムなグラフを生成するためのよく知られたErdos-Renyi法 [26] のDAG（有向非巡回グラフ）生成バリアントを使用して合成されました。Erdos-Renyi法は、パラメータ$(n,p)$が与えられると、各辺が存在する独立確率$p$を持つ$n$頂点のグラフを生成します。私たちはこの方法を修正して、目標となる辺の数を持つ_有向非巡回_グラフを生成しました。具体的には、

* DAGの頂点数$n$、頂点の最大WCETパラメータ$w$、および所望の辺の数$e$が指定されます。DAGがスケジュールされるプロセッサの数$m$も指定されます。
* 各頂点には、範囲$(1,w)$の間でランダムかつ一様に選ばれた整数としてWCETパラメータが割り当てられます。
* パラメータ$p$は、各辺の存在確率を示しており、次のように計算されます: $p \leftarrow \frac{2e}{n \times (n-1)}$。考え方としては、$n$の頂点を持つDAGは、最大で$n \times (n-1) / 2$の辺を持つので、各辺を独立して確率$p$で作成すると、平均して所望の辺数$e$が作成されるはずです。
* すべての辺は、インデックスの低い頂点から高い頂点への向きを持つと仮定されます（したがって、DAGのトポロジカルソートは、インデックスの増加順に頂点を生成できます）。各辺は、次のようにして確率$p$で作成されます: i := 1からnまで、j := (i+1)からnまで、確率pで辺(i,j)を作成します。
* 生成されたDAGのworkパラメータとspanパラメータは計算され、各頂点が正確にそのWCETパラメータ値で実行されると仮定しています（つまり、WCETパラメータは実際の実行期間を示すものとして、実行期間の上限としてではなく取られます）。これらの値を使用して、指定されたプロセッサ数$m$のDAGに対するExpression 9によるmakespanの下限と、Expression 1による上限が計算されます。
* 標準的なリストスケジューリングの実装を使用して、$m$プロセッサ上のDAGのスケジュールが生成されます（再び、各頂点が正確にそのWCETパラメータ値で実行されると仮定しています）。結果として得られるスケジュールのmakespanが記録されます。
* 表1に報告されている各データポイントは、100のこのようなグラフを生成し、それぞれの上で報告されたパラメータを計算し、その平均を取ることで得られました。

## 6 Summary and Conclusions
実時間スケジューリング理論のコミュニティで非常に人気があるDAGベースのモデルを使用して、並列可能なリアルタイムコードを表現する方法が広まってきましたが、いくつかの短所があり、特定の種類のリアルタイムコードを表現するうえでその有用性を制限しています。本論文では、代替モデルを探求しています。このモデルは、タスクをわずか2つのパラメーター、すなわちworkとspanで特徴付けるものであり、各パラメータの値の上限に対して2つの推定値を持っています。一つは非常に大きいかもしれませんが、非常に高い確度で信頼性があり、もう一つは典型的または名目上の振る舞いをより反映した小さな値です。私たちは、このようにモデル化されたタスクを、専用のプロセッサクラスタ上でスケジューリングするためのアルゴリズムを開発しました。このアルゴリズムは、_正確性_を保証する方法で動作します。つまり、実行時の動作が高信頼性の境界を超えない限り、デッドラインは常に満たされます。一方、_効率性_を追求しています。多くのプロセッサはほとんどの時間、スリープモードのままであり、実行時の動作が通常の境界を超える稀な状況でのみオンに切り替えられます。

本論文で提案されている並列化可能なコードを表現するモデル、および関連する実行時スケジューリングアルゴリズムは、特定の種類のリアルタイムアプリケーションに特に適しています。すなわち、外部環境を繰り返し（周期的にまたは散発的に）監視して、特定の種類の異常な感覚入力を検出しようとするアプリケーションです。ほとんどの場合、求められている入力は検出されず、それほど多くの計算は必要ありません。しかし、異常な入力が検出される稀な機会には、そのような入力のかなりの追加処理が必要です。さらに、その処理は高度に並列的な性質を持っています。この種の例としては、リアルタイムの侵入検出、ビジョンベースの監視システムなどがあります。このようなシステムでは、$work_N$ と $span_N$ パラメータで表される名目上のワークロードはかなり小さく、あまり並列性を持っていないことが予想されます。しかし、監視条件が発生した場合の「オーバーロード」時のワークロードはかなり集中的で、$work_O$ は大きいですが、かなりの並列性を示しています（すなわち、$span_O$ は $work_O$ に比べて相対的に小さい。同等に、$work_O$/$span_O$ の比率は大きい）。システムがハードリアルタイムである場合、正確性を保証するためのリソースの割り当ては、最悪の場合の仮定の下で行われる必要があります - $work_O$ と $span_O$ パラメータを使用して。しかし、これらの割り当てられたリソースの多くは、実行時のほとんどの時間で未使用のままになります。私たちのアプローチにより、これらの未使用のリソースが実行時に正確にいつ必要になるかを適切に判断することで、これらのリソースを必要なときまでスリープモードにしておくことができます。

私たちがここでの主要な貢献と考えるのは、並列タスクのモデルです。ランタイムスケジューラーは、このモデルを採用することのリソース効率の面での潜在的な利点を示す証拠として提示されています。今後の研究として、さまざまな環境下でのモデルの適用可能性を実証する予定です。例えば、グローバルEDFやグローバル固定優先度など、異なるスケジューリングパラダイムの下での実証を考えています。また、タスクの実行時の動作の追加のプロファイリングデータが利用可能である（そして信頼性があると知られている）場合に、タスクモデルをさらに拡張する作業も進めています。例えば、直接的な一般化により、異なる確率しきい値での複数のパラメータ値のセットを指定することが可能です（ただし2つの値のセットだけでなく）。このような追加情報を意味ある方法で活用できるスケジューリング戦略を開発することは可能でしょうか？

## References

* [1] Karthik Lakshmanan, Shinpei Kato, and Ragunathan Rajkumar. Scheduling parallel real-time tasks on multi-core processors. In _RTSS_, pages 259-268. IEEE Computer Society, 2010.
* [2] Bjorn Andersson and Dionisio de Niz. Analyzing global-edf for multiprocessor scheduling of parallel tasks. In Roberto Baldoni, Paola Flocchini, and Ravindran Binosy, editors, _Principles of Distributed Systems_, volume 7702 of _Lecture Notes in Computer Science_, pages 16-30. Springer Berlin Heidelberg, 2012.
* [3] Sanjoy Baruah, Vincenzo Bonifaci, Alberto Marchetti-Spaccamela, Leem Stougie, and Andreas Wiese. A generalized parallel task model for recurrent real-time processes. In _Proceedings of the IEEE Real-Time Systems Symposium_, RTSS 2012, pages 63-72, San Juan, Puerto Rico, 2012.
* [4] Sanjoy Baruah, Marko Bertogna, and Giorgio Buttazzo. _Multiprocessor Scheduling for Real-Time Systems_. Springer Publishing Company, Incorporated, 2015.
* [5] Jose Fonseca, Vincent Nelis, Gurulingesh Raravi, and Luis Miguel Pinho. A Multi-DAG model for real-time parallel applications with conditional execution. In _Proceedings of the ACM/ SIGAPP Symposium on Applied Computing (SAC)_, Salamanca, Spain, April 2015. ACM Press.
* [6] Alessandra Melani, Marko Bertogna, Vincenzo Bonifaci, Alberto Marchetti-Spaccamela, and Giorgio Buttazzo. Response-time analysis of conditional DAG tasks in multiprocessor systems. In _Proceedings of the 2014 26th Euromicro Conference on Real-Time Systems_, ECRTS '15, pages 222-231, Lund (Sweden), 2015. IEEE Computer Society Press.
* [7] Sanjoy Baruah, Vincenzo Bonifaci, and Alberto Marchetti-Spaccamela. The global EDF scheduling of systems of conditional sporadic DAG tasks. In _Proceedings of the 2014 26th Euromicro Conference on Real-Time Systems_, ECRTS '15, pages 222-231, Lund (Sweden), 2015. IEEE Computer Society Press.
* [8] Robert I. Davis, Alan Burns, and David Griffin. On the meaning of pWCET distributions and their use in schedulability analysis. In _Proceedings 2017 Real-Time Scheduling Open Problems Seminar (RTSOPS)_, 2017.

* [9] Alan Burns and Robert Davis. Mixed-criticality systems: A review (9th edition). [http://www-users.cs.york.ac.uk/~burns/review.pdf](http://www-users.cs.york.ac.uk/~burns/review.pdf) (Accessed on Aug 29, 2017), 2017.
* [10] Steve Vestal. Preemptive scheduling of multi-criticality systems with varying degrees of execution time assurance. In _Proceedings of the Real-Time Systems Symposium_, pages 239-243, Tucson, AZ, December 2007. IEEE Computer Society Press.
* [11] Sanjoy Baruah. The federated scheduling of systems of conditional sporadic dag tasks. In _Proceedings of the 15th International Conference on Embedded Software (EMSOFT)_, Amsterdam, the Netherlands, 2015.
* [12] James Anderson, Sanjoy Baruah, and Bjoern Brandenburg. Multicore operating-system support for mixed criticality. In _Proceedings of the Workshop on Mixed Criticality: Roadmap to Evolving UAV Certification_, San Francisco, CA, April 2009.
* [13] Alan Burns and Sanjoy Baruah. Towards a more practical model for mixed criticality systems. In _Proceedings of the International Workshop on Mixed Criticality Systems (WMC)_, December 2014.
* [14] Alan Burns and Robert I. Davis. A survey of research into mixed criticality systems. _ACM Comput. Surv._, 50(6):82:1-82:37, November 2017.
* [15] R. L. Graham, E. L. Lawler, J. K. Lenstra, and A. H. G. Rinnooy Kan. Optimization and approximation in deterministic sequencing and scheduling: A survey. _Ann. Discrete Mathematics_, 5:287-326, 1979.
* 393, 1975.
* [17] R. Graham. Bounds on multiprocessor timing anomalies. _SIAM Journal on Applied Mathematics_, 17:416-429, 1969.
* [18] J. K. Lenstra and A. H. G. Rinnooy Kan. Complexity of scheduling under precedence constraints. _Operations Research_, 26(1):22-35, 1978.
* [19] Ola Svensson. Conditional hardness of precedence constrained scheduling on identical machines. In _Proceedings of the 42nd ACM symposium on Theory of computing_, STOC '10, pages 745-754, New York, NY, USA, 2010. ACM.
* [20] S. Edgar and A. Burns. Statistical analysis of WCET for scheduling. In _2001 IEEE Real-Time Systems Symposium (RTSS)_, pages 215-224, Dec 2001.
* [21] G. Bernat, A. Colin, and S. M. Petters. WCET analysis of probabilistic hard real-time systems. In _2002 IEEE Real-Time Systems Symposium (RTSS)_, pages 279-288, 2002.
* [22] G. Bernat, A. Colin, and S. Petters. pWCET: A tool for probabilistic worst-case execution time analysis of real-time systems. Technical report, The University of York, England, 2003.
* [23] Jing Li, David Ferry, Shaurya Ahuja, Kunal Agrawal, Christopher Gill, and Chenyang Lu. Mixed-criticality federated scheduling for parallel real-time tasks. In _Proceedings of the 22nd IEEE Real-Time and Embedded Technology and Applications Symposium (RTAS)_, April 2016.
* [24] Jing Li, Abusayeed Saifullah, Kunal Agrawal, Christopher Gill, and Chenyang Lu. Analysis of federated and global scheduling for parallel real-time tasks. In _Proceedings of the 2012 26th Euromicro Conference on Real-Time Systems_, ECRTS '14, Madrid (Spain), 2014. IEEE Computer Society Press.
* [25] M. Chisholm, B. Ward, N. Kim, and J. Anderson. Cache sharing and isolation tradeoffs in multicore mixed-criticality systems. In _Real-Time Systems Symposium (RTSS), 2015 IEEE_, pages 305-316, Dec 2015.
* [26] P. Erdos and A. Renyi. On random graphs I. _Publicationes Mathematicae Debrecen_, 6:290, 1959.