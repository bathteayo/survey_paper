Under the federated paradigm of multiprocessor scheduling, a set of processors is reserved for the exclusive use of each real-time task. If tasks are characterized very conservatively (as is typical in safety-critical systems), it is likely that most invocations of the task will have computational demand far below the worst-case characterization, and could have been scheduled correctly upon far fewer processors than were assigned to it assuming the worst-case characterization of its runtime behavior. Provided we could safely determine during run-time when all the processors are going to be needed, for the rest of the time the unneeded processors could be idled in low-energy “sleep” mode, or used for executing non-real time work in the background. In this paper we propose a model for representing parallelizable real-time tasks in a manner that permits us to do so. Our model does not require us to have fine-grained knowledge of the internal structure of the code represented by the task; rather, it characterizes each task by a few parameters that are obtained by repeatedly executing the code under different conditions and measuring the run-times.

マルチプロセッサ・スケジューリングの連合パラダイムの下では、プロセッサのセットは各リアルタイム・タスクの排他的使用のために予約される。タスクが（セーフティクリティカルなシステムで典型的なように）非常に保守的にキャラクタライズされる場合、タスクのほとんどの呼び出しは、ワーストケースのキャラクタライズをはるかに下回る計算要求である可能性が高く、タスクの実行時動作のワーストケースのキャラクタライズを仮定して割り当てられたよりもはるかに少ないプロセッサで正しくスケジューリングされる可能性がある。すべてのプロセッサが必要になるタイミングをランタイム中に安全に決定できれば、残りの時間、不要なプロセッサは低エネルギーの「スリープ」モードでアイドル状態にするか、バックグラウンドで非リアルタイムの作業を実行するために使用することができる。本稿では、並列化可能なリアルタイム・タスクを表現するためのモデルを提案する。このモデルは、タスクによって表現されるコードの内部構造に関する詳細な知識を必要としない。むしろ、異なる条件下でコードを繰り返し実行し、実行時間を測定することによって得られるいくつかのパラメータによって、各タスクを特徴付ける。

Scheduling theory is concerned with the analysis of real-time systems. As multiprocessor and multicore implementations of real-time systems become prevalent, it is desirable that the models used in scheduling theory for representing real-time workloads be capable of exposing the parallelism that may exist within these workloads. This need has given rise to formal task models such as the fork-join model [1, 2], the sporadic DAG tasks model [3] (see [4, Chapter 21] for a text-book description), the multi-DAG model [5], the conditional DAG tasks_ model [6, 7] etc. Each of these models represents the internal structure of the piece of code being modeled at a relatively fine level of granularity, with the parallelism in the code typically modeled as a directed acyclic graph (DAG). Each vertex in such a DAG represents a segment of sequential code, and edges represent precedence constraints between such code segments: the segment of sequential code represented by the vertex at the tail of an edge much complete execution before the segment of sequential code represented by the vertex at the head of the edge may begin to execute.

スケジューリング理論は、リアルタイムシステムの解析に関係する。リアルタイムシステムのマルチプロセッサやマルチコアの実装が普及するにつれて、リアルタイム作業負荷を表現するスケジューリング理論で使用されるモデルは、これらの作業負荷内に存在する可能性のある並列性を明らかにすることができることが望まれている。この必要性から、フォークジョインモデル[1、2]、スポラディックDAGタスクモデル[3]（教科書的な説明は[4、21章]を参照）、マルチDAGモデル[5]、条件付きDAGタスク_モデル[6、7]などの正式なタスクモデルが生まれた。これらのモデルはそれぞれ、モデル化されるコードの内部構造を比較的細かい粒度で表し、コード内の並列性は通常、有向無サイクルグラフ（DAG）としてモデル化される。このようなDAGの各頂点はシーケンシャルコードのセグメントを表し、エッジはそのようなコードセグメント間の優先順位の制約を表す。エッジの末尾の頂点で表されるシーケンシャルコードのセグメントは、エッジの先頭の頂点で表されるシーケンシャルコードのセグメントが実行を開始する前に、実行を完了する必要がある。

Such DAG-based models for representing parallel real-time code have proved popular in the real-time scheduling theory community, and much important and interesting research has been accomplished that is based upon representing systems using these models. This body of research has indeed provided us with a deeper insight into the issues that arise in exploiting parallelism in multiprocessor real-time systems; however due to a variety of reasons (some of which are enumerated and discussed in some detail in Section 2) there are some classes of real-time applications for which such DAG-based representations may not be appropriate for the purposes of schedulability analysis; alternative representations are needed. In this paper we propose one such possible alternative representation that may be suitable under certain circumstances. In this model we do not attempt to explicitly represent the internal parallel structure of the code. Instead, we seek to identify a few important parameters of parallalizable code that are most useful for scheduling algorithms that seek to schedule such code upon multiprocessor platforms, and propose that the code be looked upon as a "black box" that is characterized by just these parameters. Furthermore, we do not require that the internal structure of the code be examined in order to obtain these parameter values. Rather, we propose that values for these parameters be estimated via extensive simulation experiments: repeatedly executing the code in a controlled laboratory environment in order to be able to compute bounds on the parameter values. (Such an approach is inspired by the large and growing body of current research [8] on probabilistic worst-case execution time - pWCET - analysis.) Since measurement-based approaches are typically not able to provide parameter values that are guaranteed correct with absolute certainty, we incorporate, from the mixed-criticality scheduling literature [9], Vestal's idea [10] of characterizing a single task with two sets of parameters: one set very conservative and hence trusted to a very high level of assurance and the other, far less conservative but more representative of "typical" behavior.

並列リアルタイムコードを表現するためのこのようなDAGベースのモデルは、リアルタイムスケジューリング理論のコミュニティで人気があり、これらのモデルを使用してシステムを表現することに基づく多くの重要で興味深い研究が達成されてきた。しかし、様々な理由（その幾つかは列挙され、セクション2で詳細に議論される）により、このようなDAGベースの表現がスケジューラビリティ解析の目的に適さないリアルタイムアプリケーションのクラスが存在し、代替の表現が必要とされている。この論文では、特定の状況下で適切な代替表現の1つを提案する。このモデルでは、コードの内部並列構造を明示的に表現しようとはしない。その代わりに、マルチプロセッサ・プラットフォーム上で並列化可能なコードをスケジューリングしようとするスケジューリング・アルゴリズムにとって最も有用な、並列化可能なコードのいくつかの重要なパラメータを特定し、コードをこれらのパラメータだけで特徴付けられる「ブラックボックス」として見なすことを提案する。さらに、これらのパラメータ値を得るためにコードの内部構造を調べる必要はない。むしろ、これらのパラメータの値は、大規模なシミュレーション実験によって推定することを提案する。パラメータ値の境界を計算できるようにするために、制御された実験室環境でコードを繰り返し実行するのだ。 (このようなアプローチは、確率論的ワーストケース実行時間pWCET解析に関する現在の研究[8]の大規模かつ成長過程に触発されたものである）。測定に基づくアプローチは、通常、絶対確実な正しさが保証されるパラメータ値を提供することができないので、混合クリティカリティスケジューリング文献[9]から、Vestalのアイデア[10]を取り入れ、2つのパラメータセットで1つのタスクを特徴付ける。

Organization.The remainder of this paper is organized as follows. In Section 2 we motivate the new model by identifying relevant characteristics of parallelizable real-time code that current models are not well-suited to represent, and formally define the workload and system model that we are proposing. In Section 3 we briefly discuss some prior research that provides the foundations upon which our proposed model is built. In Section 4 we derive, and prove the correctness and other relevant properties of, an algorithm for scheduling systems represented using the proposed model. Our overall objective is to be able to obtain more resource-efficient implementations of systems, while ensuring correctness; in Section 5 we explore some possible means of further enhancing the efficiency of the algorithm presented in Section 4. We conclude in Section 6 with a discussion on the relevance, significance, and limitations of our proposed model, and an enumeration of possible directions for continued research.

本論文の残りの部分は、以下のように構成されている。セクション2では、現在のモデルが表現するのに適していない並列化可能なリアルタイムコードの関連する特性を特定することで、新しいモデルの動機付けを行い、我々が提案するワークロードとシステムモデルを正式に定義する。セクション3では、提案するモデルの基礎となる先行研究について簡単に述べる。セクション4では、提案モデルを用いて表現されるシステムのスケジューリング アルゴリズムを導出し、その正しさとその他の関連する特性を証明する。我々の全体的な目的は、正しさを保証しつつ、よりリソース効率の良いシステムの実装を得ることである。セクション5では、セクション4で提示したアルゴリズムの効率をさらに向上させるために可能ないくつかの手段を検討する。セクション6では、提案モデルの妥当性、重要性、限界について議論し、今後の研究の方向性を列挙する。
 

## 2 System model: Motivation and Definition

In this section we flesh out the details of the model we are proposing for representing parallelizable real-time code that is not conveniently represented using previously-proposed DAG-based task models. We will first motivate the model informally, and seek to explain aspects of the model via illustrative examples. A formal definition of the model is then provided in Section 2.1; our proposed algorithm for scheduling tasks represented using this model is described in Section 2.2.

このセクションでは、これまで提案されてきたDAGベースのタスクモデルでは表現しにくい、並列化可能なリアルタイムコードを表現するために提案するモデルの詳細を説明する。まず、モデルの動機付けを非公式に行い、例示を通してモデルの側面を説明する。その後、セクション2.1でモデルの正式な定義を行い、セクション2.2では、このモデルを使って表現されたタスクをスケジューリングするためのアルゴリズムを提案する。

Why a new model?As stated in Section 1 above, several excellent DAG-based models for representing parallel real-time code have been developed in the real-time scheduling theory community; however there are some classes of real-time applications for which such models have proved unsuitable. This may be for one or more of the following reasons:

なぜ新しいモデルなのか？上記セクション1で述べたように、並列リアルタイムコードを表現するためのいくつかの優れたDAGベースのモデルがリアルタイムスケジューリング理論のコミュニティで開発されてきた。しかし、そのようなモデルが適さないことが判明したリアルタイムアプリケーションのクラスがいくつか存在する。これは、以下のような理由によるものである：

1. The internal structure of the parallel code may be very complex, with multiple conditional dependencies (as may be represented in e.g., the conditional DAG tasks model [6, 7]) and (bounded) loops. Explicit enumeration of all possible paths through such code in order to identify worst-case behavior may be computationally infeasible.3 Footnote 3: We point out that techniques for _approximating_ the worst-case behavior of complex conditional parallelizable code have been proposed with regards to specific scheduling algorithms such as global fixed-priority [6], global EDF [7] or federated [11].
1. 並列コードの内部構造は、（条件付きDAGタスクモデル[6,7]などで表現されるような）複数の条件付き依存関係や（境界付き）ループなど、非常に複雑な場合があります。注3：複雑な条件付き並列化可能コードのワーストケース動作を近似する技術は、グローバル固定優先[6]、グローバルEDF[7]、フェデレート[11]などの特定のスケジューリングアルゴリズムに関して提案されている。
2. If some parts of the code are procured from outside the application-developers' organization, the provider of this code may seek to protect their intellectual property (IP) by not revealing the internal structure of the code and instead only providing executables - this may be the case if, e.g., commercial vision algorithms are used in a real-time application. (Although reverse-engineering of the executable code in order to determine its internal structure may be possible in principle, such reverse engineering tends to be tedious and error-prone.)
2. コードの一部がアプリケーション開発者の組織外から調達されている場合、このコードの提供者は、コードの内部構造を明らかにせず、代わりに実行可能コードのみを提供することによって、知的財産（IP）を保護しようとするかもしれません。(内部構造を特定するために実行コードをリバースエンジニアリングすることは原理的には可能かもしれませんが、そのようなリバースエンジニアリングは面倒でエラーが発生しがちです)。
3. Algorithms for the analysis of systems represented using DAG-based models tend to have run-time pseudo-polynomial or exponential in the size of the DAG. Such run-times have traditionally been considered acceptably small enough to allow the algorithms to be practical in practice; however, this state of affairs may not continue in the future. For many cyber-physical real-time systems, constraints such as deadlines are typically dictated by physical factors. As the processors upon which we implement such cyber-physical real-time systems become increasingly more powerful, it becomes possible to incorporate far more complex processing that would be represented as larger DAGs than was previously the case. As this trend towards more complex processing and the consequent larger DAGs continues, run-times pseudo-polynomial in the size of these larger DAGs may become too large be used in practice during system design and analysis.
3. DAGベースのモデルを用いて表現されるシステムの解析のためのアルゴリズムは、DAGのサイズに対して擬似多項式または指数関数的な実行時間を持つ傾向がある。このような実行時間は、従来、アルゴリズムが実際に実用的であるのに十分許容できるほど小さいと考えられてきたが、この状態が将来も続くとは限らない。多くのサイバーフィジカル・リアルタイムシステムでは、納期などの制約は一般的に物理的要因によって決定される。このようなサイバーフィジカル・リアルタイム・システムを実装するプロセッサがますます強力になるにつれて、従来よりもはるかに複雑な処理を組み込むことが可能になり、より大きなDAGとして表現されるようになる。より複雑な処理とそれに伴うより大きなDAGへのこの傾向が続くと、システム設計や解析の際に、これらの大きなDAGのサイズに擬似的に多項式となるランタイムが大きくなりすぎて、実際に使用できなくなる可能性がある。
4. Further exacerbating the situation, explicitly representing the internal structure of some pieces of parallel code in DAG form results in DAGs that may be of size exponential in the size of the code. Consider, for example, the following code snippet written in OpenMP ([http://www.openmp.org/](http://www.openmp.org/)), an application programming interface (API) that supports multi-platform shared memory multiprocessing programming: #pragma omp parallel #pragma omp for for (i=0; i<10; i++) { //do_something }

This code snippet would translate to a DAG with $(1+10+1=)$ 12 nodes. If we were to replace the "10" in the upper bound of the for loop with a "100", however, the resulting DAG would have 102 nodes; replacing it with "1000" would yield 1002 nodes, etc. - increasing the size of the program by one ASCII character results in an almost ten-fold increase in the size of the DAG.

4. さらに状況を悪化させるのは、並列コードの一部の内部構造をDAG形式で明示的に表現すると、DAGのサイズがコードのサイズの指数関数になる可能性があることです。例えば、マルチプラットフォーム共有メモリ・マルチプロセッシング・プログラミングをサポートするアプリケーション・プログラミング・インターフェース（API）であるOpenMP（[http://www.openmp.org/](http://www.openmp.org/)）で書かれた以下のコードスニペットを考えてみましょう： #pragma omp parallel #pragma omp for (i=0; i<10; i++) { //何かする }.

このコード・スニペットは、$(1+10+1=)$ 12ノードを持つDAGに変換される。しかし、forループの上限の "10 "を "100 "に置き換えると、結果のDAGは102ノードになり、"1000 "に置き換えると1002ノードになる。- プログラムのサイズをASCII文字1文字増やすと、DAGのサイズはほぼ10倍になる。

5. Particularly for conditional code, it may be the case that the true worst-case behavior of the code is very infrequently expressed during run-time.4. Traditional models based on conditional DAGs may not be suitable for representing such code (although mixed-criticality [10, 12, 13, 14] extensions of such conditional DAG models are a possibility - to our knowledge, such models have not yet been proposed, let alone studied). Footnote 4: Consider, for example, a real-time application that periodically monitors a sensor for anomalous input. Most of the time the sought-for anomolous input is not detected, and not much computation needs to be performed. But on the rare occasions when anomalous input is detected, considerable additional processing of such input is necessary. For pieces of parallel real-time code possessing one or more of the characteristics discussed above, DAG-based representations may not be appropriate for the purposes of schedulability analysis; alternative representations are needed. Let us now discuss what such a representation should provide.

5. 特に条件付きコードの場合、実行時にコードの真のワーストケース動作が表現されることは非常にまれである。4.条件付きDAGに基づく従来のモデルは、このようなコードを表現するのに適していない可能性がある（ただし、このような条件付きDAGモデルの混合クリティカル[10, 12, 13, 14]拡張は可能性があるが、我々の知る限り、このようなモデルはまだ提案されておらず、研究されていない）。脚注4：例えば、センサーの異常入力を定期的に監視するリアルタイムアプリケーションを考えてみよう。ほとんどの場合、異常な入力は検出されず、計算を実行する必要はあまりありません。しかし、まれに異常入力が検出された場合、そのような入力に対してかなりの追加処理が必要になる。上述したような特徴を1つ以上持つ並列リアルタイム・コードの場合、DAGベースの表現はスケジューラビリティ解析の目的には適さないかもしれない。それでは、そのような表現が提供すべきものについて説明しよう。

Identifying relevant characteristics of parallelizable real-time code.In modeling parallelizable real-time code that is to be executed upon a multiprocessor platform, a prime objective is to enable the exploitation of the parallelism that may be present in the code by scheduling algorithms, in order to enhance the likelihood that we will be able to meet timing constraints. We are interested here in developing _predictable_ real-time systems - systems that can have their timing (and other) correctness verified prior to run-time. For the purposes of enabling a priori timing verification, decades of research in the parallel computing community suggests the following two timing parameters of a piece of parallelizable code are particularly significant:

マルチプロセッサ・プラットフォーム上で実行される並列化可能なリアルタイム・コードをモデリングする際の主な目的は、タイミング制約を満たす可能性を高めるために、スケジューリング・アルゴリズムによってコードに存在する可能性のある並列性を利用できるようにすることである。我々がここで興味を持っているのは、予測可能なリアルタイム・システム、つまり実行前にタイミング（およびその他の）正しさを検証できるシステムの開発である。先験的なタイミング検証を可能にするために、並列コンピューティングコミュニティにおける数十年にわたる研究は、並列化可能なコードの以下の2つのタイミングパラメータが特に重要であることを示唆している：

1. The **work** parameter denotes the cumulative worst-case execution time of all the parallel branches that are executed across all processors. Note that for non-conditional parallelizable code this is equal to the worst case execution time of the code on a single processor (ignoring communication overhead from synchronizing processors).

1. **work**パラメータは、全プロセッサで実行されるすべての並列分岐のワーストケースの累積実行時間を示す。非条件付き並列化可能なコードでは、これは単一プロセッサ上でのコードのワーストケース実行時間に等しいことに注意してください（プロセッサの同期による通信オーバーヘッドを無視します）。

2. The **span** parameter denotes the maximum cumulative worst-case execution time of any sequence of precedence-constrained pieces of code. It represents a lower bound on the duration of time the code would take to execute, regardless of the number of processors available. The span of a computation is also called the _critical path length_ of the computation, and a sequence of precedence-constrained pieces of code with cumulative worst-case execution time equal to the span, a _critical path_ through the computation. The relevance of these two parameters arises from well-known results in scheduling theory concerning the multiprocessor scheduling of precedence-constrained jobs (i.e., DAGs) to minimize makespan - this is the widely-studied P$|$ pre$|$$\mathcal{S}_{\max}$ problem in the classic 3-field $\alpha\mid\beta\mid\gamma$ notation that is commonly used in scheduling theory [15]. This problem has long been known to be NP-hard in the strong sense [16]; i.e., computationally highly intractable. However, Graham's _list scheduling_ algorithm [17], which constructs a work-conserving schedule by executing at each instant in time an available job, if any are present, upon any available processor, performs fairly well in practice. It was shown [17] that list scheduling makes the following guarantee: if $\mathcal{S}_{\max}$ denotes the minimum makespan with which a particular DAG can be scheduled upon $m$ processors, then the schedule generated by list scheduling this DAG upon $m$ processors will have a makespan no greater than $(2-\frac{1}{m})\times\mathcal{S}_{\max}$. This result, in conjunction with a hardness result in [18] showing that determining a schedule for this DAG of makespan $\leq\frac{4}{3}\mathcal{S}_{\max}$ remains NP-hard in the strong sense5, suggests that list scheduling is a reasonable algorithm to use in practice, and in fact most run-time scheduling algorithms that are used for scheduling DAGs upon multiprocessors use some variant or the other of list scheduling. We will do so in this paper as well.

2. **span**パラメータは、優先順位に制約のあるコード・シーケンスの最大累積ワース ト・ケース実行時間を示す。これは、利用可能なプロセッサの数に関係なく、コードの実行にかかる時間の下限を示す。計算のスパンは、計算の_クリティカルパスの長さ_とも呼ばれ、累積最悪実行時間がスパンに等しい優先順位制約付きコード片のシーケンスは、計算を通る_クリティカルパス_となる。これら2つのパラメータの関連性は、優先度に制約されたジョブ(すなわち、DAG)のマルチプロセッサ・スケジューリングが、メイクスパンを最小化することに関するスケジューリング理論のよく知られた結果から生じる。これは、スケジューリング理論で一般的に使用される古典的な3-field $alpha=alpha=mid=beta=mid=gamma$記法における、広く研究されているP$|$ pre$|$mathcal{S}_{max}$問題である[15]。
この問題は長い間、強い意味でNP困難であることが知られている [16]。しかし、Grahamの_list scheduling_ アルゴリズム [17]は、利用可能なジョブが存在する場合、利用可能なプロセッサ上で各瞬間に実行することにより、仕事を保存するスケジュールを構築するものであり、実際にはかなり良好に動作する。リストスケジューリングが以下の保証をすることが示された[17]： $mathcal{S}_{max}$ が特定のDAGを$m$個のプロセッサでスケジューリングできる最小のメー クスパンを示すとすると、このDAGを$m$個のプロセッサでリストスケジューリングして 生成されるスケジュールは、$(2-frac{1}{m})˶timesmathcal{S}_{max}$より大きいメー クスパンを持たない。この結果は、[18]における、このDAGのスケジューリングがメー クスパン$leqfrac{4}{3}}のNP-hardであることを示すhardnessの結 果と合わせて、リストスケジューリングが実際に使用する妥当なアルゴリズム であることを示唆しており、実際、マルチプロセッサ上のDAGのスケジューリン グに使用されるほとんどのランタイムスケジューリングアルゴリズムは、 何らかのリストスケジューリングの変形を使用している。本稿でもそうする。

Footnote 5: In fact, assuming a reasonable complexity-theoretic conjecture that is somewhat stronger than $\mathrm{P}\neq\mathrm{NP}$, a result of Svensson [19] implies that a polynomial-time algorithm for determining a schedule of makespan $\leq 2\mathcal{S}_{\max}$ for all $m$ is ruled out.

An upper bound on the makespan of a schedule generated by list scheduling is easily stated. Letting work and span denote the work and span parameters of the DAG being scheduled, it has been proved in [17] that the makespan of the schedule for a given DAG is guaranteed to be no larger than

$\frac{\mathrm{work}-\mathrm{span}}{m}+\mathrm{span} $

リストスケジューリングによって生成されるスケジュールのメークスパンの上界は簡単に記述できる。workとspanをスケジューリングされるDAGのworkとspanパラメータとすると、与えられたDAGに対するスケジュールのmakespanが以下の値以下であることが保証されることが[17]で証明されている。

$\frac{\mathrm{work}-\mathrm{span}}{m}+\mathrm{span}$ (1)

Thus a good upper bound on the makespan of the list-scheduling generated schedule for a DAG may be stated in terms of only its work and span parameters. Equivalently if the DAG represents a real-time piece of code characterized by a relative deadline parameter $D$, $(\frac{\mathrm{work}-\mathrm{span}}{m}+\mathrm{span})\leq D$ is a sufficient test for determining whether the code will complete by its deadline upon an $m$-processor platform. _We therefore identify the work and span parameters of a piece of parallel real-time code as being particularly relevant from the perspective of schedulability analysis._

従って、DAGのリストスケジューリング生成スケジュールのメー クスパンの良い上限は、workパラメータとspanパラメータだけで記述できる。 従って、並列リアルタイムコードのworkパラメータとspanパラメータは、スケジューリング可能性解析の観点から特に関連性が高いことを確認する_。

A measurement-based approach to parameter estimation.The work and span parameters of tasks that are represented using DAG-based models are quite straightforward to compute in time linear in the representation of the DAGs (algorithms for doing so are described in [3, 7]). As we have discussed above, however, our interest is in characterizing parallel code that is typically not conveniently represented using DAG-based models. We propose that for the purposes of representing pieces of such code for schedulability analysis, we _ignore_ their internal structure and instead seek to characterize them solely via their work and span parameters. And since we cannot in general determine the precise values of the work and span parameters of a piece of code without knowing its internal structure, we advocate here that _measurement-based_ approaches be used to _estimate_ these parameters. Measurement-based approaches have been developed for estimating probabilistic worst-case execution time (pWCET) [20, 21, 22] distributions of individual pieces of code, and implemented in pWCET tools such as RapiTime ([https://www.rapidasystems.com/products/rapitime](https://www.rapidasystems.com/products/rapitime)) from Rapita Systems. We now briefly discuss how measurement-based approaches may be adapted to estimate the work and span parameters of parallel code. Ignoring overhead associated with implementing global scheduling, observe that the work parameter of a piece of code is equal to the time needed to complete its execution upon a single processor, while its span parameter is equal to the time needed to complete its execution upon an unbounded number of processors. Hence one can estimate the probability distribution of the work parameter of a piece of code by using pWCET techniques to estimate its WCET distribution upon a single processor. One can similarly estimate the probability distribution of the span parameter by 1. first adapting the measurement-based techniques underpinning pWCET-estimation to determine the makespan probability distribution upon a given number of processors (rather than the completion-time upon a single processor); and then
2. estimating the makespan distribution of the parallel code upon platforms in which the number of processors is repeatedly increased, until further increases do not result in significant changes to the estimated distribution.

DAGベースのモデルを用いて表現されるタスクの作業パラメータとスパンパラメータは、DAGの表現に線形な時間で計算するのが非常に簡単である（そのためのアルゴリズムは[3, 7]に記述されている）。しかし、上述したように、我々の興味は、一般的にDAGベースのモデルを使って表現するのが便利ではない並列コードを特徴付けることにある。スケジューラビリティ解析のためにこのようなコード片を表現する目的で、その内部構造を無視し、その代わりに、workパラメータとspanパラメータのみによってコード片を特徴付けることを提案する。そして、一般的に、コードの内部構造を知らなければ、コード片のworkパラメータとspanパラメータの正確な値を決定することができないので、ここでは、これらのパラメータを推定するために、測定ベースのアプローチを使用することを提唱する。測定ベースのアプローチは、個々のコード片の確率的ワーストケース実行時間（pWCET）[20, 21, 22]分布を推定するために開発され、Rapita Systems社のRapiTime（[https://www.rapidasystems.com/products/rapitime](https://www.rapidasystems.com/products/rapitime)）などのpWCETツールに実装されている。
ここで、並列コードのworkパラメータとspanパラメータを推定するために、測定に基づくアプローチをどのように適応できるかを簡単に説明する。グローバルスケジューリングの実装に伴うオーバーヘッドを無視すると、コードのworkパラメータは1つのプロセッサで実行を完了するのに必要な時間に等しく、spanパラメータは無限のプロセッサで実行を完了するのに必要な時間に等しいことがわかる。従って、pWCET技術を用いて単一プロセッサ上でのWCET分布を推定することで、コード片の作業パラメータの確率分布を推定することができる。同様にスパンパラメータの確率分布も、1.まずpWCET推定を支える計測ベースの技術を適応させ、（単一プロセッサでの完了時間ではなく）与えられたプロセッサ数でのメイクスパンの確率分布を決定する。
2.プロセッサ数が繰り返し増加するプラットフォームで並列コードの処理時間分布を推定する。


The proposed model: multiple work and span estimates.As discussed above, it is possible, by suitable adaptation and application of pWCET techniques, to estimate probability distributions for the work and span parameters of pieces of parallel real-time code. It is understood in the pWCET community that pWCET-based techniques cannot in general determine bounds that are guaranteed to be correct with absolute certainty; rather, they provide bounds that are guaranteed correct to specified probabilistic degrees of confidence/ levels of assurance (Davis et al. [8] provide a thoughtful and considered discussion as to how the concept of probabilities should be interpreted when used in such a manner). In the model we propose for representing parallel real-time code, we suggest that each such piece of code be characterized by _two_ pairs of (work, span) parameter values, each pair corresponding to a different probability threshold in the work and span distributions and therefore valid at different levels of assurance. Specifically, one pair of values should be _very_ conservative and therefore trusted to a very high level of assurance and the other, while still relatively safe, should be more representative of "typical" behavior, not attempting to cover scenarios that are highly unlikely to occur. We illustrate via an example.

上述したように、pWCET技術を適切に適応・適用することで、並列リアルタイムコード片のworkパラメータとspanパラメータの確率分布を推定することが可能である。pWCET コミュニティでは、pWCET ベースの技法は、一般に、絶対的に正しいことが保証された境界を決定することはできない。むしろ、指定された確率的な信頼度／保証レベルまで正しいことが保証された境界を提供することが理解されている（Davis ら [8]は、このような方法で使用する場合に、確率の概念をどのように解釈すべきかについて、思慮深く、考慮された議論を提供している）。並列リアルタイム・コードを表現するために提案するモデルでは、各コードを2組の（work, span）パラメータ値で特徴付けることを提案する。各組はwork分布とspan分布の異なる確率閾値に対応し、したがって異なる保証レベルで有効である。具体的には、1組の値は_非常に_保守的であり、したがって非常に高い保証レベルで信頼されるべきであり、もう1組は、まだ比較的安全ではあるが、より「典型的な」動作を代表するものであるべきであり、発生する可能性が非常に低いシナリオをカバーしようとしてはならない。例を挙げて説明しよう。


**Example 1**.: Suppose that we were able to determine for a piece of code that

* Its work parameter is $>120$ with some small probability $p$, but $>900$ with a far smaller probability $p^{\prime}\ll p$.
* Its span parameter is $>40$ with probability $p$; however the probability that it is $>600$ is $p^{\prime}$.

We could characterize this piece of code with two ordered pairs of (work,span) values - a $(1-p)$ probability of being $\leq(120,40)$, and a far greater $(1-p^{\prime})$ probability of being $\leq(900,600)$.

We require that _correctness criteria hold under the more conservative estimate_. Suppose for instance that it were specified that this code should execute within a relative deadline of $D$; we require that the makespan by $\leq D$ provided $\text{work}\leq 900$ and $\text{span}\leq 600$.

**例 1**: あるコードの特性を以下のように判定できると仮定しましょう。

- その作業パラメータが確率 $p$ で $>120$ であり、確率 $p^{\prime}\ll p$ で $>900$ である。
- そのスパンパラメータが確率 $p$ で $>40$ であり、確率 $p^{\prime}$ で $>600$ である。

このコードを、2つの順序付けられた (作業, スパン) の組み合わせで特徴づけることができます。それは、$(1-p)$ の確率で $(120, 40)$ 以下であることを示し、さらに $(1-p^{\prime})$ の確率で $(900, 600)$ 以下であることを示します。

このコードに関して、より保守的な見積もりの下で**正確性の基準**が満たされる必要があります。たとえば、このコードが相対的な締め切り $D$ 内で実行されることが指定されている場合、作業が $900$ 以下かつスパンが $600$ 以下である限り、Makespan（最大実行時間）が $D$ 以下である必要があります。

The proposed run-time scheduling approach.Since correctness is defined with respect to the more conservative estimates for work and span, in order to satisfy correctness requirements we must provision computing resources to a task assuming these more conservative estimates. However, it is our expectation that the task's run-time behavior is very likely to be bounded by the less conservative parameter estimates, and hence statically provisioning adequate resources for it under the more conservative assumptions is likely to result in significant wastage of computing resources during run-time. One manner of ameliorating such wastage is by keeping some of the provisioned resource in "reserve", perhaps by placing some processors in sleep mode or having them execute background (non real-time) work, with the option of switching them to work upon executing the task if we determine, during run-time, that the task's run-time behavior is in fact not likely to be bounded by the less conservative parameter estimates. The following example illustrates.

提案するランタイムスケジューリングアプローチ。正しさは、より保守的なワークとスパンの推定値に関して定義されるので、正しさの要求を満たすためには、より保守的な推定値を仮定してタスクに計算資源を提供しなければならない。しかし、タスクの実行時動作は、より保守的でないパラメータ推定値によって制限される可能性が非常に高いため、より保守的な仮定の下で静的に適切なリソースを提供することは、実行時に計算リソースを大幅に浪費する可能性が高い。そのような浪費を改善する1つの方法は、プロビジョニングされたリソースの一部を「リザーブ」にしておくことです。おそらく、いくつかのプロセッサをスリープモードにするか、バックグラウンド（非リアルタイム）作業を実行させます。次の例はそれを示している。

**Example 2**.: Suppose the code in Example 1 to be scheduled with a relative deadline equal to 690, upon a 10-processor platform. According to Expression 1, the makespan of the schedule upon 10 processors is no more than

$\frac{900-600}{10}+600=(30+600)=\mathbf{630}$

assuming the more conservative work and span estimates hold. Hence, correctness is guaranteed.

**例 2**: 例 1のコードが、相対的な締め切りが 690 である 10 プロセッサのプラットフォームでスケジュールされると仮定します。式 1に従うと、10 プロセッサ上でのスケジュールのMakespan（最大実行時間）は以下のように計算されます。

$\frac{900-600}{10}+600=(30+600)=\mathbf{630}$

より保守的な作業およびスパンの見積もりが成立すると仮定すると、したがって、正確性が保証されます。

この計算により、10プロセッサのプラットフォームでのMakespanが最大で630であるため、相対的な締め切り690を満たすことが確認されます。

In fact, ten processors are not necessary for correctness: if we were executing this piece of code upon just four processors, the corresponding makespan bound according to Expression 1 would be $\left(\frac{900-600}{4}\right)+600=\mathbf{675}$. Our run-time algorithm could therefore realize some energy savings by simply switching off six of the ten provided processors, and executing the task on the remaining four.

実際には、正確性を保証するためには10プロセッサは必要ありません。このコードを4つのプロセッサで実行する場合、式1に従って得られるMakespanの制約は次のように計算できます：

$\left(\frac{900-600}{4}\right)+600=\mathbf{675}$

したがって、提供された10プロセッサのうち6つを単にオフにして、残りの4つでタスクを実行することで、ランタイムアルゴリズムはエネルギーの節約を実現できるでしょう。

Could we switch off seven processors? The reader may verify that with three processors Expression 1would yield a makespan bound of $\left(\frac{900-600}{3}\right)+600=\mathbf{700}$. Since 700 exceeds the specified relative deadline of 690, we conclude that we may miss the deadline if we were to switch off seven processors and the system behaved worse than anticipated by its less conservative parameters (although not its more conservative parameters). We therefore conclude that we need at least $\mathbf{4}$ processors to not be in sleep mode, in order to ensure correctness.

7つのプロセッサをオフにできるでしょうか？3つのプロセッサで実行した場合、Expression 1によって計算されるMakespanの制約は次のようになります：

$\left(\frac{900-600}{3}\right)+600=\mathbf{700}$

700は指定された相対的な締め切りである690を超えるため、7つのプロセッサをオフにした場合、システムがそのより保守的なパラメータではなく、より保守的なパラメータによって予想されるよりも悪く振る舞う可能性がある場合、締め切りを逃す可能性があることが確認されます。

したがって、正確性を確保するためには、スリープモードにならないように少なくとも4つのプロセッサが必要であると結論できます。

The run-time algorithm we will derive in this paper is designed for systems in which the less conservative parameters are very likely to hold "most of the time"; i.e., the value of $p$ is itself very small. Provided such is the case for this example

* our run-time scheduling algorithm starts out scheduling the system on just three processors, leaving the remaining seven processors in sleep mode.
* If execution has not completed by some time-instant (whose value is precomputed), it wakes up the sleeping processors and makes all ten processors available for this task to execute upon.

We will prove later that with this algorithm, the task completes by the specified deadline of 690 provided the more conservative task parameters hold; **correctness** is thus established. Additionally, there is a $\leq p$ probability that it will not complete by the pre-computed time-instant and hence need to awaken the remaining seven processors.

この論文で説明されているランタイムアルゴリズムは、より保守的なパラメータが「ほとんどの場合」成立すると非常に確率が高いシステム向けに設計されています。つまり、$p$ の値自体が非常に小さいと仮定されています。この例においても、この前提が成り立つ場合を考えてみましょう。

* ランタイムスケジューリングアルゴリズムは、最初にシステムを3つのプロセッサでスケジュールし、残りの7つのプロセッサをスリープモードにします。
* 実行がある事前に計算された時間に完了していない場合、スリープモードのプロセッサを起動し、このタスクを実行するために10つのプロセッサをすべて使用可能にします。

このアルゴリズムによれば、より保守的なタスクパラメータが成立する場合、タスクは指定された締め切りである690までに完了し、**正確性**が確立されます。さらに、事前計算された時間に完了しない可能性が $\leq p$ であり、したがって残りの7つのプロセッサを起動する必要がある可能性があります。

このアルゴリズムにより、より保守的なパラメータがほとんどの場合成立するシステムに対して、正確性を確保しながらエネルギーを節約することができます。

To evaluate the **efficiency**, suppose, for this example, that $p=0.05$, indicating that the less conservative parameters hold with 95% probability. There is therefore a $\leq 5\%$ probability that the task will not complete by the specified time-instant,6 and the expected number of processors that would be needed is no larger than

$\left(0.95\times 3+0.05\times 10\right)=\left(2.85+0.5\right)=\mathbf{3.35}$

in contrast to the four that would be needed if our run-time scheduling algorithm were to not be used. 

**効率性**を評価するために、この例において $p=0.05$ と仮定しましょう。これは、より保守的なパラメータが95%の確率で成立することを示しています。したがって、指定された時間にタスクが完了しない確率は $\leq 5\%$ であり、必要なプロセッサの期待値は次のように計算できます：

$\left(0.95\times 3+0.05\times 10\right)=\left(2.85+0.5\right)=\mathbf{3.35}$

この場合、我々のランタイムスケジューリングアルゴリズムを使用しない場合に必要とされる4つのプロセッサと比べて、必要なプロセッサ数は3.35となります。

このアルゴリズムを使用することで、より保守的なパラメータがほとんどの場合成立する場合に、タスクを効率的に実行するために必要なプロセッサ数を減らすことができ、効率性を向上させることができます。

Footnote 6: In fact, since the work and span parameters will not in general be perfectly correlated, the expected probability of this happening is likely to be far less than 5%. (This issue is revisited in Section 5.)

脚注6: 実際には、作業パラメータとスパンパラメータは一般的に完全に相関しないため、この事が発生する予測確率は5%未満である可能性が高いです。この問題についてはセクション5で再考されます。

Examples 1 and 2 above have illustrated the task model, and the associated run-time strategy for scheduling tasks that are so modeled, that we are proposing in this paper. We now formally define the task model in Section 2.1, and the run-time scheduler in Section 2.2, below.

上記の例1と例2は、この論文で提案されているタスクモデルと、そのモデルに基づいてスケジュールされるタスクのランタイム戦略を示しています。次に、我々はセクション2.1でタスクモデルを正式に定義し、セクション2.2でランタイムスケジューラを定義します。

### System Model

We now provide a formal definition of our model, by describing in detail the workload model we assume. The workload we seek to model comprises a single piece of parallelizable real-time code that is characterized by the following list of parameters:


以下は、モデルを正式に定義するための記述で、我々が想定しているワークロードモデルについて詳細に説明しています。我々がモデル化しようとしているワークロードは、以下のパラメータのリストによって特徴づけられる、並列化可能なリアルタイムコードの1つからなります。

$<work_O, span_O, work_N, span_N, D>$

with the following interpretation:

1. $(\operatorname{\mathrm{work}}_{O},\operatorname{\mathrm{span}}_{O})$. These represent very conservative estimates of the true "worst-case" values of the work and span parameters; as discussed above, we expect that these estimates will be obtained using the kinds of measurement-based techniques that have been developed for estimating probabilistic worst-case execution time (pWCET) distributions.
1. $(\operatorname{\mathrm{work}}_{O}, \operatorname{\mathrm{span}}_{O})$。これらは作業パラメータとスパンパラメータの真の「最悪の場合」の値の非常に保守的な見積もりを表します。先に述べたように、これらの見積もりは、確率的最悪実行時間（pWCET）分布を推定するために開発された測定ベースの技術を使用して得られると予想されています。

2. $D$ denotes the _relative deadline_ parameter: for correct execution it is required that the job be scheduled with makespan no greater than $D$. We highlight here that timing correctness is specified assuming that the $(\operatorname{\mathrm{work}}_{O},\operatorname{\mathrm{span}}_{O})$ parameter estimates are correct: the code is required to complete execution within the specified relative deadline $D$ provided its work and span parameters are no larger than $\operatorname{\mathrm{work}}_{O}$ and $\operatorname{\mathrm{span}}_{O}$ respectively.

2. $D$ は **相対的な締め切り** パラメータを示しています。正確な実行のためには、ジョブがMakespanが $D$ を超えないようにスケジュールされる必要があります。ここで強調するのは、タイミングの正確性は、$(\operatorname{\mathrm{work}}_{O}, \operatorname{\mathrm{span}}_{O})$ パラメータの推定が正しいと仮定して指定されていることです。コードは、その作業パラメータとスパンパラメータがそれぞれ $\operatorname{\mathrm{work}}_{O}$ および $\operatorname{\mathrm{span}}_{O}$ 以下である場合に、指定された相対的な締め切り $D$ 内で実行が完了する必要があります。

3. $(\operatorname{\mathrm{work}}_{N},\operatorname{\mathrm{span}}_{N})$. These are less conservative estimates on the values of the work and span parameters: it is expected that the actual values of the work and span parameters are _very_ likely to be no larger than $\operatorname{\mathrm{work}}_{N}$ and $\operatorname{\mathrm{span}}_{N}$ respectively. (The subscript "$N$" in $\operatorname{\mathrm{work}}_{N}$ and $\operatorname{\mathrm{span}}_{N}$ stand for "nominal" [23].) These parameter estimates play no role in defining correctness; rather (as we have seen in Examples 1 and 2), their values may be used for the purposes of devising more resource-efficient scheduling strategies. It is hence not as critical that their values be assigned correctly as it is for the $\operatorname{\mathrm{work}}_{O}$ and $\operatorname{\mathrm{span}}_{O}$ parameters: while incorrectly estimated values for $\operatorname{\mathrm{work}}_{O}$ and $\operatorname{\mathrm{span}}_{O}$ may compromise timing correctness in the sense that we may end up missing deadlines, incorrectly estimated values for $\operatorname{\mathrm{work}}_{N}$ and $\operatorname{\mathrm{span}}_{N}$ simply result in less efficient implementations. We assume that $\operatorname{\mathrm{work}}_{N}\leq\operatorname{\mathrm{work}}_{O}$ and $\operatorname{\mathrm{span}}_{N}\leq\operatorname{\mathrm{span}}_{O}$. (Although our results are readily extended to situations where these assumptions do not hold, we do not see a rationale for relaxing these assumptions, since by very definition the $(\operatorname{\mathrm{work}}_{N},\operatorname{\mathrm{span}}_{N})$ parameters represent less conservative estimates than the $(\operatorname{\mathrm{work}}_{O},\operatorname{\mathrm{span}}_{O})$ parameters.)

3. $(\operatorname{\mathrm{work}}_{N}, \operatorname{\mathrm{span}}_{N})$。これらは作業パラメータとスパンパラメータの値に対するより保守的でない見積もりです。実際の作業パラメータとスパンパラメータが、$\operatorname{\mathrm{work}}_{N}$ および $\operatorname{\mathrm{span}}_{N}$ 以下である可能性が**非常に高い**と予想されています（$\operatorname{\mathrm{work}}_{N}$ と $\operatorname{\mathrm{span}}_{N}$ のサブスクリプト "$N$" は "nominal" [23] の略です）。これらのパラメータの推定値は、正確性を定義する役割を果たしません。代わりに（例1および例2で示されているように）、これらの値はリソース効率の高いスケジューリング戦略を考案するために使用される可能性があります。そのため、$\operatorname{\mathrm{work}}_{N}$ および $\operatorname{\mathrm{span}}_{N}$ パラメータの値が正確に割り当てられる必要性は、$\operatorname{\mathrm{work}}_{O}$ および $\operatorname{\mathrm{span}}_{O}$ パラメータの場合ほど重要ではありません。$\operatorname{\mathrm{work}}_{O}$ と $\operatorname{\mathrm{span}}_{O}$ の誤った推定値は、締め切りを逃す可能性があるという意味でタイミングの正確性を損なう可能性がありますが、$\operatorname{\mathrm{work}}_{N}$ と $\operatorname{\mathrm{span}}_{N}$ の誤った推定値は単に効率の低い実装になる結果となります。我々は $\operatorname{\mathrm{work}}_{N}\leq\operatorname{\mathrm{work}}_{O}$ および $\operatorname{\mathrm{span}}_{N}\leq\operatorname{\mathrm{span}}_{O}$ であると仮定します（これらの仮定が成立しない場合でも、我々の結果は容易に拡張できますが、$\operatorname{\mathrm{work}}_{N}$ および $\operatorname{\mathrm{span}}_{N}$ パラメータが $\operatorname{\mathrm{work}}_{O}$ および $\operatorname{\mathrm{span}}_{O}$ パラメータよりも保守的でないという定義上の理由から、これらの仮定を緩和する理由は見当たりません）。

In this paper, we consider the scheduling of a single such task upon a dedicated bank of identical processors. We point out that our results are directly applicable to the scheduling of recurrent - _periodic_ or _sporadic_ - real-time DAGs under the federated paradigm [24] of multiprocessor scheduling, provided each periodic/ sporadic task satisfies the additional constraint that its relative deadline parameter is no larger than its period parameter (i.e., they are _constrained-deadline_ tasks). We believe our approach is particularly appropriate for scheduling systems of recurrent tasks: for such tasks, we anticipate that the $(\operatorname{\mathrm{work}}_{N},\operatorname{\mathrm{span}}_{N})$ parameters will bound the behavior of most invocations ("dag-jobs" [3]) of the task, with an occasional rare dag-job exceeding these bounds. Hence many of the allocated processors will remain in sleep mode most of the time, with the occasional dag-job requiring that the sleeping processors be awakened until that dag-job completes execution, after which they can be returned to sleep mode.

この論文では、同一のプロセッサバンクに対する単一のタスクのスケジューリングに焦点を当てています。また、私たちの結果は、マルチプロセッサのスケジューリングの「連邦パラダイム」[24]の下での定期的またはスポラディックなリアルタイムDAG（有向非循環グラフ）のスケジューリングにも直接適用できます。前述の条件を満たす限り、各定期的/スポラディックなタスクがその相対的な締め切りパラメータがその周期パラメータよりも大きくない制約つき締め切りタスクである場合です。我々は、我々のアプローチが定期的なタスクのスケジューリングシステムに特に適していると考えています。このようなタスクの場合、$(\operatorname{\mathrm{work}}_{N},\operatorname{\mathrm{span}}_{N})$ パラメータが、そのタスクのほとんどの呼び出し（"dag-jobs" [3]とも呼ばれる）の動作を制約すると予想しており、まれな場合にはこれらの制約を超えることがあります。したがって、割り当てられた多くのプロセッサは、ほとんどの時間スリープモードにとどまり、まれにスリープモードのプロセッサを起動する必要があるまで、スリープモードにとどまるでしょう。その後、そのdag-jobの実行が完了したら、プロセッサを再びスリープモードに戻すことができます。

### The Scheduling Algorithm

Given a task as specified above:

$<\operatorname{work}_{O},\operatorname{span}_{O},\operatorname{ work}_{N},\operatorname{span}_{N},D >$

that is to be executed upon a platform comprising $m$ identical processors, we first perform some **pre-run-time** schedulability analysis that determines whether we are able to schedule the task upon the $m$ processors in a manner that ensures correctness. Recall that _correctness_ is specified as requiring that the task meet its deadline provided its work parameter is $\leq\operatorname{work}_{O}$ and its span parameter is $\leq\operatorname{span}_{O}$: by Expression 1, this is guaranteed provided

$(\frac{\operatorname{work}_{O}-\operatorname{span}_{O}}{m}+\operatorname {span}_{O})\leq D;$ 

上記で指定されたタスク：

$<\operatorname{work}{O}, \operatorname{span}{O}, \operatorname{work}{N}, \operatorname{span}{N}, D >$

このタスクを $m$ 個の同一のプロセッサからなるプラットフォームで実行する場合、まず実行前のスケジュラビリティ分析を行います。この分析により、タスクを $m$ 個のプロセッサで正確性を保証する方法でスケジュールできるかどうかが決定されます。ここで、正確性は、タスクがその作業パラメータが $\leq \operatorname{work}{O}$ であり、スパンパラメータが $\leq \operatorname{span}{O}$ の場合に締め切りを満たすことを要求するものです。Expression 1によれば、これは次の条件を満たす場合に保証されます：

$(\frac{\operatorname{work}_{O}-\operatorname{span}_{O}}{m}+\operatorname {span}_{O})\leq D;$ 

この条件が成立する場合、タスクは指定された相対締め切り $D$ 内で正確に実行されることが保証されます。

If Condition 2 does not hold, our scheduling algorithm declares failure: it is unable to schedule this instance in a manner that guarantees timing correctness. Otherwise, it computes a pair of values $m_{N}$ and $\mathcal{S}_{N}$ - the manner in which these values are computed will be derived in Section 4.2. These computed parameters have the following intended interpretation: provided the less conservative work and span parameter estimates are correct (i.e., work is $\leq\operatorname{work}_{N}$ and span is $\leq\operatorname{span}_{N}$ for the task), list scheduling can schedule the task upon $m_{N}$ processors to have a makespan no greater then $\mathcal{S}_{N}$.

条件2が成立しない場合、スケジューリングアルゴリズムは失敗と宣言します。これは、このインスタンスをタイミングの正確性を保証する方法でスケジュールできないことを意味します。それ以外の場合、アルゴリズムは $m_{N}$ および $\mathcal{S}{N}$ という2つの値を計算します。これらの値の計算方法はセクション4.2で導出されます。計算されたこれらのパラメータは、次のように解釈されます。より保守的でない作業とスパンパラメータの推定が正確である場合（つまり、タスクの作業が $\leq \operatorname{work}{N}$ であり、スパンが $\leq \operatorname{span}{N}$ である場合）、リストスケジューリングは、タスクを $m{N}$ のプロセッサ上で $\mathcal{S}_{N}$ を超えないMakespanでスケジュールできることを意味します。

Run-time scheduling.Suppose that the piece of parallelizable real-time code represented by this task is activated at some time-instant $t_{o}$ during run-time.

1. The scheduler sets a timer to go off at time-instant $(t_{o}+\mathcal{S}_{N})$, and begins executing the task upon $m_{N}$ processors using the list-scheduling algorithm [17]. The remaining $(m-m_{N})$ processors assigned to this task are placed/ remain in sleep mode.
2. If the task has not completed execution by time-instant $(t_{o}+\mathcal{S}_{N})$, then the scheduler awakens the $(m-m_{N})$ sleeping processors, and uses list-scheduling to execute the remainder of the task upon the entire bank of $m$ processors.
3. As mentioned in Section 2.1 above, in the case of recurrent tasks these awakened processors are returned to sleep mode upon completion of execution of the current dag-job of the task.

実行時のスケジューリング：このタスクで表される並列化可能なリアルタイムコードが、実行時のある時間インスタント $t_{o}$ にアクティブ化されたと仮定します。

1. スケジューラはタイマーを設定し、時間インスタント $(t_{o}+\mathcal{S}_{N})$ でタイマーが作動するようにし、タスクを $m_{N}$ 個のプロセッサ上でリストスケジューリングアルゴリズム [17] を使用して実行を開始します。このタスクに割り当てられた残りの $(m-m_{N})$ 個のプロセッサはスリープモードに配置またはスリープモードに留まります。
2. タスクが時間インスタント $(t_{o}+\mathcal{S}_{N})$ までに実行が完了しない場合、スケジューラは $(m-m_{N})$ 個のスリーププロセッサを起動し、タスクの残りの部分を全ての $m$ 個のプロセッサのバンクでリストスケジューリングを使用して実行します。
3. セクション2.1で述べたように、定期的なタスクの場合、これらの起動されたプロセッサは現在のdag-jobの実行が完了すると、スリープモードに戻されます。

この方法により、タスクの効率的なスケジューリングが実現され、少ないプロセッサで効率的に実行できる場合にはスリープモードに保持され、必要に応じて起動されます。

In Section 4 we will derive the manner in which the values of $m_{N}$ and $\mathcal{S}_{N}$ are to be computed in order to guarantee correctness: the algorithm completes execution of the task within $D$ time units of its arrival, provided its work parameter is $\leq\operatorname{work}_{O}$ and its span parameter is $\leq\operatorname{span}_{O}$.

We close this section with an example illustrating the operation of our run-time scheduler.

セクション4では、正確性を保証するために $m_{N}$ および $\mathcal{S}_{N}$ の値がどのように計算されるかについて説明します。このアルゴリズムにより、タスクの作業パラメータが $\leq\operatorname{work}_{O}$ であり、スパンパラメータが $\leq\operatorname{span}_{O}$ の場合、タスクは到着から $D$ 時間単位以内に実行が完了します。

このセクションを閉じる前に、私たちの実行時スケジューラの動作を説明する例を示します。

Example3 Consider once again the instance discussed in Examples 1 and 2. In the notation of Section 2, this task is represented by the following parameters:

$\operatorname{work}_{O},\operatorname{span}_{O}, \operatorname{work}_{N},\operatorname{span}_{N},D = 900,600,120,40,690$

It is to be scheduled upon $m=10$ processors. In Example 4 we will show that the algorithm of Section 4 assigns the parameter $m_{N}$ the value $3$, and the parameter $\mathcal{S}_{N}$, a value $66\frac{2}{3}$. Hence our run-time scheduler starts out scheduling this task on $3$ processors. If the task
behaves as specified by its workN, spanN parameters, then by Expression 1 the makespan is no more than
$(120 − 40)/3 + 40 = 26*2/3 + 40 = 66*2/3$
and hence the additional seven processors are not needed. If it does not complete by timeinstant $66*2/3$ , all ten processors become available for this task to execute upon, and results in Section 4 allow us to conclude that the task does execute correctly, completing by the specified deadline at time-instant 690.

Example 3 再び例1および例2で議論されたインスタンスを考えてみましょう。セクション2の表記法では、このタスクは次のパラメータで表されます：

$\operatorname{work}_{O}, \operatorname{span}_{O}, \operatorname{work}_{N}, \operatorname{span}_{N}, D = 900, 600, 120, 40, 690$

このタスクは $m=10$ 個のプロセッサでスケジュールされます。Example 4では、セクション4のアルゴリズムがパラメータ $m_{N}$ に値 $3$ を割り当て、パラメータ $\mathcal{S}_{N}$ に値 $66\frac{2}{3}$ を割り当てることを示します。したがって、私たちの実行時スケジューラはこのタスクを最初に $3$ 個のプロセッサでスケジュールします。タスクがその workN, spanN パラメータによって指定されたように振る舞う場合、Expression 1によれば、Makespan は以下のようになります：

$
\left(\frac{120 - 40}{3} + 40\right) = 26\frac{2}{3} + 40 = 66\frac{2}{3}
$

したがって、追加の7つのプロセッサは必要ありません。タスクが $66\frac{2}{3}$ の時間インスタントまでに完了しない場合、すべての10個のプロセッサがこのタスクの実行に利用可能となり、セクション4の結果により、タスクは指定された締め切り時間である時間インスタント690までに正しく実行されることが確認できます。

## 3 Related Work
The approach to the modeling and run-time scheduling of parallelizable tasks that we are proposing here draws inspiration from research in the areas of parallel computing, mixedcriticality scheduling, and probabilistic WCET. As stated in Section 2 above, the problem of scheduling DAGs to minimize makespan (the P| prec| Smax problem in 3-field notation [15]) has been very widely studied in “traditional” scheduling theory. Given the inherent intractability of this problem [16] and the existence of a good approximation (as represented by List Scheduling [17] with its associated makespan bound – Inequality 1), the parallel computing community soon began to focus upon the work and span parameters as reasonable proxies for parallelizable computational workloads; this is one of the fundamental ideas that underpins our proposed approach. The concept of specifying multiple values, which are considered trustworthy to different levels of assurance, to a task’s parameters was proposed by Vestal [10] and forms the basis of mixed-criticality scheduling theory. There is a large body of research exploring the Vestal model – see [14] for a survey. In studying the scheduling of mixed-criticality parallel tasks, Li et al. [23] first proposed a model in which each task is characterized by different work and span parameters at low and high criticality levels – it is this model that we are studying in depth here. (The overall context of the research in [23] is quite different from ours: while we are, in the terminology of [23], considering the scheduling of a single parallelizable real-time task with the objective of minimizing the number of processors used in the nominal case while concurrently guaranteeing to meet deadlines in the overloaded case, [23] was concerned with devising mixed-criticality scheduling algorithms with good capacity augmentation bounds.) Our approach also draws upon ideas from the considerable body of prior research (e.g., [20, 21, 22]) on measurement-based techniques for estimating probabilistic worstcase execution time distributions (pWCET). The correctness of our scheduling framework very strongly depends upon the validity and accuracy of pWCET-estimation techniques, since we are in effect guaranteeing correct timing behavior (meeting deadlines) under the assumption that the more conservative estimations – workO and spanO – are correct upper bounds. In contrast, incorrect estimations of workN and spanN do not compromise correctness, although they could have an adverse impact on efficiency. Rather than being considered as estimations of worst-case parameter values, these parameters are perhaps closer in spirit to what Chisholm et al [25] have called provisioned parameter values and Li et al. [23], nominal parameter values – values that represent typical or common-case behavior and may be obtained by, e.g., somewhat inflating average-case parameter values.

私たちがここで提案している並列可能なタスクのモデリングと実行時スケジューリングのアプローチは、並列計算、混合クリティカリティスケジューリング、および確率的WCETに関する研究からインスピレーションを得ています。セクション2で述べたように、DAG（Directed Acyclic Graphs）をスケジュールしてMakespanを最小化する問題（3つのフィールド表記でのP| prec| Smax問題[15]）は、「伝統的な」スケジューリング理論で非常に広く研究されてきました。この問題の計算の難しさ[16]と、良い近似解（不等式1で表されるMakespanの境界を持つList Scheduling [17]など）の存在に鑑みて、並列計算コミュニティはすぐに、並列可能な計算ワークロードの合理的なプロキシとしての作業とスパンパラメータに焦点を当てるようになりました。これは、私たちの提案されたアプローチの基盤となる基本的なアイデアの1つです。タスクのパラメータに異なる信頼度レベルで信頼性の異なる複数の値を指定するという概念は、Vestal [10]によって提案され、混合クリティカリティスケジューリング理論の基盤となっています。Vestalモデルを探究する大量の研究が存在し、サーベイについては[14]を参照してください。混合クリティカリティ並列タスクのスケジューリングを研究する中で、Liら[23]は、各タスクが低クリティカルなレベルと高クリティカルなレベルで異なる作業とスパンパラメータで特徴付けられるモデルを最初に提案しました。これが、私たちがここで詳細に研究しているモデルです。ただし、[23]の研究の全体的な文脈は私たちのものとはかなり異なります。私たちは、[23]の用語で言えば、通常のケースで使用するプロセッサの数を最小化することを目的として、過負荷の場合に締め切りを満たすことを同時に保証する、単一の並列可能なリアルタイムタスクのスケジューリングを考えています。[23]は、容量増加の境界が良好な混合クリティカリティスケジューリングアルゴリズムを考案することに焦点を当てていました。私たちのアプローチは、確率的最悪実行時間分布（pWCET）の推定に関する測定ベースの技術に関する先行研究（例：[20, 21, 22]）からのアイデアも取り入れています。私たちのスケジューリングフレームワークの正確性は、pWCET推定技術の妥当性と精度に非常に依存しています。なぜなら、より保守的な推定値であるworkOとspanOが正確な上限であると仮定した場合、正確なタイミング動作（締め切りの満たすこと）を保証しているからです。一方、workNとspanNの不正確な推定値は正確性に影響を及ぼさず、効率に悪影響を及ぼす可能性があります。これらのパラメータは、最悪の場合のパラメータ値の推定ではなく、おそらくChisholmら[25]が提供した「プロビジョニングされたパラメータ値」と呼ばれるものに近い性格を持っており、Liら[23]の「名義的なパラメータ値」と呼ばれるものに近い性格を持っています。これらの値は、典型的なケースや一般的なケースの振る舞いを表し、平均ケースのパラメータ値をいくらか膨らませることで得られる可能性があります。


## 4 Scheduling Algorithm Derivation and Analysis

Given a task characterized, as described in Section 2.1, by the parameters

$\text{work}_{O},\text{span}_{O},\text{work}_{N},\text{span}_{N},D$

and $m$ processors upon which to execute it, we discuss in this section how we should compute values of $m_{N}$ and $\mathcal{S}_{N}$ in order to ensure that the run-time scheduling algorithm described in Section 2.2 above completes execution of the task within $D$ time units of its arrival. We will start out in Section 4.1 assuming that values for $m_{N}$ and $\mathcal{S}_{N}$ are already known, and derive sufficient conditions for ensuring timing correctness given these values of $m_{N}$ and $\mathcal{S}_{N}$. We will then describe, in Sections 4.2, how values may be assigned to $m_{N}$ and $\mathcal{S}_{N}$ in a manner that ensures that these sufficient conditions are satisfied.

セクション2.1で説明されたように、パラメータ

$\text{work}_{O},\text{span}_{O},\text{work}_{N},\text{span}_{N},D$

によって特徴付けられるタスクが与えられ、それを実行するための $m$ 個のプロセッサがある場合、このセクションでは、タスクの到着から $D$ 時間単位以内に実行を完了させるために $m_{N}$ および $\mathcal{S}_{N}$ の値をどのように計算すべきかについて説明します。セクション4.1では、$m_{N}$ および $\mathcal{S}_{N}$ の値が既にわかっていると仮定し、これらの値を使用してタイミングの正確性を確保するための十分な条件を導出します。次に、セクション4.2で、$m_{N}$ および $\mathcal{S}_{N}$ に値を割り当てる方法について説明し、これらの十分な条件が満たされるようにします。


### Sufficient Schedulability Conditions

Suppose that we are given values of $m_{N}$ and $\mathcal{S}_{N}$ (with $0<m_{N}\leq m$ and $0\leq\mathcal{S}_{N}\leq D$), and the run-time algorithm schedules the task on $m_{N}$ processors using list scheduling. If the task completes execution within $\mathcal{S}_{N}$ time units, correctness is preserved since $\mathcal{S}_{N}\leq D$. It remains to determine sufficient conditions for correctness when the task does not complete by time-instant $\mathcal{S}_{N}$; this we do in the remainder of this section.

Figure 1 depicts the processors that are available for this task if it does _not_ complete execution within $\mathcal{S}_{N}$ time units, thereby resulting in the run-time scheduler awakening the $(m-m_{N})$ processors that had been in sleep mode over $[0,\mathcal{S}_{N})$. We will now derive conditions for ensuring that the task completes execution by its deadline at time-instant $D$ when executing upon these available processors, given that its work parameter may be as large as $\text{work}_{O}$ and its span parameter, $\text{span}_{O}$.

$m_{N}$ と $\mathcal{S}_{N}$ の値が与えられた場合（$0<m_{N}\leq m$ および $0\leq\mathcal{S}_{N}\leq D$）、実行時アルゴリズムはリストスケジューリングを使用してタスクを $m_{N}$ 個のプロセッサでスケジュールします。タスクが $\mathcal{S}_{N}$ 時間単位以内に実行を完了する場合、$\mathcal{S}_{N}\leq D$ であるため、正確性は維持されます。タスクが $\mathcal{S}_{N}$ の時間インスタントまでに完了しない場合の正確性を確保するための十分な条件を決定する残りの部分をこのセクションで行います。

図1は、タスクが $\mathcal{S}_{N}$ の時間単位内に実行を完了しない場合に利用可能なプロセッサを示しており、これにより実行時スケジューラが $[0,\mathcal{S}_{N})$ の間スリープモードにあった $(m-m_{N})$ 個のプロセッサを起動します。これらの利用可能なプロセッサ上で実行される場合、その作業パラメータが $\text{work}_{O}$ と同じぐらい大きいかもしれないし、スパンパラメータが $\text{span}_{O}$ と同じぐらい大きいかもしれないと仮定して、タスクが時間インスタント $D$ で締め切りを満たすための条件を導出します。

Let $\text{work}^{\prime}$ and $\text{span}^{\prime}$ denote the work and span parameters of the amount of computation of the parallel task that remains at time-instant $\mathcal{S}_{N}$ (these are $>0$, since the task is assumed to not have completed execution by time-instant $\mathcal{S}_{N}$). This remaining computation executes upon $m$ processors; By Expression 1 the overall makespan is therefore bounded from above by

$\mathcal{S}_{N}+\left(\frac{\text{work}^{\prime}-\text{span}^{\prime}}{m}+ \text{span}^{\prime}\right)$

$\text{work}^{\prime}$ および $\text{span}^{\prime}$ を、時間インスタント $\mathcal{S}_{N}$ での並列タスクの計算量の作業パラメータとスパンパラメータとして定義しましょう（これらは $>0$ です、なぜならタスクは時間インスタント $\mathcal{S}_{N}$ までに実行が完了しないと仮定しているからです）。この残りの計算は $m$ 個のプロセッサで実行されます。式1によれば、全体のメイクスパンは次のように上から制約されます。

$\mathcal{S}_{N}+\left(\frac{\text{work}^{\prime}-\text{span}^{\prime}}{m}+ \text{span}^{\prime}\right)$

Figure 1: The parallel task begins execution at time-instant $0$ with a deadline at time-instant $D$. It executes upon $m_{N}$ processors over the interval $[0,\mathcal{S}_{N})$, and upon $m$ processors over the interval $[\mathcal{S}_{N},D)$. (The $x$-axis thus denotes time, and the $y$-axis, the processors.)Since the remaining span at time-instant $\mathcal{S}_{N}$ is $\text{span}^{\prime}$, an amount $(\text{span}_{O}-\text{span}^{\prime})$ of the critical path of the task has executed during $[0,\mathcal{S}_{N}]$. At each instant when the critical path is not executing, it must be the case that all $m_{N}$ processors are busy executing tasks not on the critical path. Hence the total amount of execution occurring over $[0,\mathcal{S}_{N})$ is at least

$(\mathcal{S}_{N}-(\text{span}_{O}-\text{span}^{\prime}))\times m_{ N}+(\text{span}_{O}-\text{span}^{\prime}),$

from which it follows that

$\text{work}^{\prime} \leq \text{work}_{O}-\mathcal{S}_{N}\times m_{N}+(\text{span}_{O}- \text{span}^{\prime})\times m_{N}-(\text{span}_{O}-\text{span}^{\prime}) $ $= \text{work}_{O}-\mathcal{S}_{N}\times m_{N}+(\text{span}_{O}- \text{span}^{\prime})\times(m_{N}-1)$ $= \text{work}_{O}-\mathcal{S}_{N}\times m_{N}+\text{span}_{O}\times (m_{N}-1)-\text{span}^{\prime}\times(m_{N}-1)$

図1: 並列タスクは時間インスタント0で実行を開始し、時間インスタントDで締め切りがあります。それは $[0,\mathcal{S}_{N})$ の間で $m_{N}$ 個のプロセッサ上で実行され、 $[\mathcal{S}_{N},D)$ の間で $m$ 個のプロセッサ上で実行されます。 （したがって、x軸は時間を示し、y軸はプロセッサを示します。）時間インスタント $\mathcal{S}_{N}$ での残りのスパンは $\text{span}^{\prime}$ であり、タスクのクリティカルパスの $(\text{span}_{O}-\text{span}^{\prime})$ が $[0,\mathcal{S}_{N}]$ の間に実行されています。クリティカルパスが実行されていない各瞬間では、すべての $m_{N}$ 個のプロセッサがクリティカルパスに関係のないタスクを実行している必要があります。したがって、$[0,\mathcal{S}_{N})$ の間に発生する実行の合計は少なくとも次のようになります。

$(\mathcal{S}_{N}-(\text{span}_{O}-\text{span}^{\prime}))\times m_{N}+(\text{span}_{O}-\text{span}^{\prime})$

これから次のことがわかります。

$\text{work}^{\prime} \leq \text{work}_{O}-\mathcal{S}_{N}\times m_{N}+(\text{span}_{O}- \text{span}^{\prime})\times m_{N}-(\text{span}_{O}-\text{span}^{\prime}) $ $= \text{work}_{O}-\mathcal{S}_{N}\times m_{N}+(\text{span}_{O}- \text{span}^{\prime})\times(m_{N}-1)$ $= \text{work}_{O}-\mathcal{S}_{N}\times m_{N}+\text{span}_{O}\times (m_{N}-1)-\text{span}^{\prime}\times(m_{N}-1)$

Substituting Inequality 4 into the Expression 3, we obtain the following upper bound on the overall makespan:

$\mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)-\text{span}^{\prime}\times(m_{N}-1 )-\text{span}^{\prime}}{m}+\text{span}^{\prime}) $ $= \mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)-\text{span}^{\prime}\times m_{N}} {m}+\text{span}^{\prime})$ $= \mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)}{m}-\text{span}^{\prime}\times \frac{m_{N}}{m}+\text{span}^{\prime})$ $= \mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)}{m}+\text{span}^{\prime}\times (1-\frac{m_{N}}{m}))$

不等式4を式3に代入すると、全体のメイクスパンに対する次の上限値が得られます：

$\mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)-\text{span}^{\prime}\times(m_{N}-1 )-\text{span}^{\prime}}{m}+\text{span}^{\prime}) $ $= \mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)-\text{span}^{\prime}\times m_{N}} {m}+\text{span}^{\prime})$ $= \mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)}{m}-\text{span}^{\prime}\times \frac{m_{N}}{m}+\text{span}^{\prime})$ $= \mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)}{m}+\text{span}^{\prime}\times (1-\frac{m_{N}}{m}))$

Since $m_{N}\leq m$, Expression 5 is maximized when $\text{span}^{\prime}$ is large as possible; i.e., $\text{span}^{\prime}=\text{span}_{O}$ (the physical interpretation is that the worst case occurs when no job on the critical path is executed prior to time-instant $\mathcal{S}_{N}$: instead the entire critical path executes after $\mathcal{S}_{N}$). Substituting $\text{span}^{\prime}\leftarrow\text{span}_{O}$ into Expression 5, we get the following upper bound on the overall makespan:

$\mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)}{m}+\text{span}_{O}\times(1- \frac{m_{N}}{m}))$ (6) $= \mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}-\text{span}_{O}}{m}+\text{span}_{O})$ Correctness is guaranteed by having this upper bound on the makespan be $\leq D$: $(\mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N }\times m_{N}-\text{span}_{O}}{m}+\text{span}_{O}))\leq D$ $\Leftrightarrow (\mathcal{S}_{N}-\frac{\mathcal{S}_{N}\times m_{N}}{m} )\leq(D-\frac{\text{work}_{O}-\text{span}_{O}}{m}-\text{span}_{O} )$ $\Leftrightarrow \mathcal{S}_{N}(1-\frac{m_{N}}{m})\leq(D-\frac{ \text{work}_{O}-\text{span}_{O}}{m}-\text{span}_{O})$ Expression 6 above is thus the sufficient schedulability condition we seek: values of $m_{N}$ and $\mathcal{S}_{N}$ satisfying Expression 6 guarantee timing correctness.

$m_{N}\leq m$ であるため、式5は $\text{span}^{\prime}$ ができるだけ大きい場合に最大化されます。つまり、$\text{span}^{\prime}=\text{span}_{O}$ です（物理的な解釈は、クリティカルパス上のジョブが時間インスタント $\mathcal{S}_{N}$ の前に実行されない場合、最悪の場合が発生する：代わりにクリティカルパス全体が $\mathcal{S}_{N}$ の後に実行されます）。式5に $\text{span}^{\prime}\leftarrow\text{span}_{O}$ を代入すると、次の全体のメイクスパンの上限値が得られます：

$\mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}+\text{span}_{O}\times(m_{N}-1)}{m}+\text{span}_{O}\times(1- \frac{m_{N}}{m}))$（6）$= \mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N} \times m_{N}-\text{span}_{O}}{m}+\text{span}_{O})$。

この上限値が $D$ 以下であることにより、正確性が保証されます：$(\mathcal{S}_{N}+(\frac{\text{work}_{O}-\mathcal{S}_{N }\times m_{N}-\text{span}_{O}}{m}+\text{span}_{O}))\leq D$ $\Leftrightarrow (\mathcal{S}_{N}-\frac{\mathcal{S}_{N}\times m_{N}}{m} )\leq(D-\frac{\text{work}_{O}-\text{span}_{O}}{m}-\text{span}_{O} )$ $\Leftrightarrow \mathcal{S}_{N}(1-\frac{m_{N}}{m})\leq(D-\frac{ \text{work}_{O}-\text{span}_{O}}{m}-\text{span}_{O})$。

上記の式6は、タイミングの正確性を保証するための十分なスケジュラビリティ条件であり、式6を満たす $m_{N}$ と $\mathcal{S}_{N}$ の値は、タイミングの正確性を保証します。

### Computing $m_{N}$ and $\mathcal{S}_{N}$

We saw in Section 4.1 above that in order to ensure correctness, our scheduling algorithm should choose the parameters $m_{N}$ and $\mathcal{S}_{N}$ such that Condition 6 above is satisfied. Recall that an additional goal is _efficiency_: the smaller the value of $m_{N}$, the better, since the remaining $(m-m_{N})$ processors can be placed in sleep mode. In this section we describe how our algorithm computes such a value.

One reasonable approach for assigning a value to the $m_{N}$ parameter is by using the task's _nominal_ work and span parameters $\text{work}_{N}$ and $\text{span}_{N}$. Assuming that these parameters bound the work and span values of a "typical" invocation of the task, it is guaranteed by Inequality 1 that upon $m_{N}$ processors a typical invocation will have a makespan no greater than $((\text{work}_{N}-\text{span}_{N})/m_{N}+\text{span}_{N})$. We may hence assign $\mathcal{S}_{N}$ a value as follows:

$\mathcal{S}_{N}\leftarrow(\frac{\text{work}_{N}-\text{span}_{N}}{m_{N}} +\text{span}_{N})$

タイミングの正確性を保証するために、セクション4.1で見たように、スケジューリングアルゴリズムは条件6を満たすようにパラメーター$m_{N}$および$\mathcal{S}_{N}$を選択すべきです。追加の目標は「効率性」であり、$m_{N}$の値が小さいほど良いです。なぜなら、残りの$(m-m_{N})$プロセッサはスリープモードに配置できるからです。このセクションでは、アルゴリズムがそのような値を計算する方法を説明します。

$m_{N}$パラメーターに値を割り当てるための1つの合理的なアプローチは、タスクの「名目」の作業およびスパンパラメーター$\text{work}_{N}$および$\text{span}_{N}$を使用することです。これらのパラメータがタスクの「典型的な」呼び出しの作業およびスパン値を制限すると仮定すると、不等式1によって、$m_{N}$のプロセッサ上での典型的な呼び出しのメイクスパンは次のようになります：$((\text{work}_{N}-\text{span}_{N})/m_{N}+\text{span}_{N})$。したがって、$\mathcal{S}_{N}$に値を次のように割り当てることができます：

$\mathcal{S}_{N}\leftarrow(\frac{\text{work}_{N}-\text{span}_{N}}{m_{N}} +\text{span}_{N})$

Substituting this value for $\mathcal{S}_{N}$ into Expression 6, we get

$(\frac{\text{work}_{N}-\text{span}_{N}}{m_{N}}+\text{span}_{N}) \times(1-\frac{m_{N}}{m})\leq(D-\frac{\text{work}_{O}-\text{ span}_{O}}{m}-\text{span}_{O})$

as a sufficient schedulability condition. Since every term other than $m_{N}$ is a constant in this expression, the expression can be algebraically simplified to a form that is a quadratic expression in $m_{N}$; solving this quadratic expression, and taking the ceiling (since the number of processors $m_{N}$ must be integral) yields the desired value. Once $m_{N}$ is so computed, the value of $\mathcal{S}_{N}$ may be obtained from Expression 7. We illustrate via an example; the algorithm for computing $m_{N}$ and $\mathcal{S}_{N}$ is provided in pseudo-code form after the example.

この値を$\mathcal{S}_{N}$の式6に代入すると、次のようになります。

$(\frac{\text{work}_{N}-\text{span}_{N}}{m_{N}}+\text{span}_{N}) \times(1-\frac{m_{N}}{m})\leq(D-\frac{\text{work}_{O}-\text{ span}_{O}}{m}-\text{span}_{O})$

これは十分なスケジューラビリティ条件です。この式では、$m_{N}$以外の各項が定数であるため、この式を$m_{N}$の2次式の形に代数的に簡略化できます。この2次式を解き、天井関数を取ることで（プロセッサの数$m_{N}$は整数でなければならないため）、必要な値を得ることができます。一旦$m_{N}$が計算されたら、$\mathcal{S}_{N}$の値は式7から得ることができます。以下に例を示します。$m_{N}$および$\mathcal{S}_{N}$を計算するアルゴリズムは、この例の後に疑似コード形式で提供されています。

Consider once again the instance discussed in Examples 1 and 2:

$<\text{work}_{O},\text{span}_{O},\text{work}_{N},\text{ span}_{N},D> = <900,600,120,40,690>$

to be scheduled upon $m=10$ processors.

Substituting these values into Expression 8, we get

$(\frac{work_{N}-\text{span}_{N}}{m_{N}}+\text{span}_{N}) \times(1-\frac{m_{N}}{m})\leq(D-\frac{\text{work}_{O}-\text{ span}_{O}}{m}-\text{span}_{O})$ $\equiv (\frac{120-40}{m_{N}}+40)\times(1-\frac{m_{N}}{ 10})\leq(690-\frac{900-600}{10}-600)$ $\equiv (\frac{80}{m_{N}}+40)\times(1-\frac{m_{N}}{10} )\leq 60$ $\equiv 40\cdot(\frac{2}{m_{N}}+1)\times(1-\frac{m_{N} }{10})\leq 60$ $\equiv 2\cdot(\frac{2+m_{N}}{m_{N}})\times(\frac{10-m_ {N}}{10})\leq 3$ $\equiv (2+m_{N})\times(10-m_{N})\leq 15m_{N}$ $\equiv 20+8m_{N}-m_{N}^{2}\leq 15m_{N}$ $\equiv m_{N}^{2}+7m_{N}-20\geq 0$

from which we obtain

$m_{N}\geq\frac{-7+\sqrt{129}}{2}\ \approx\ 2.18$

もちろん、数式部分をそのまま翻訳します。

再度、Examples 1および2で議論されたインスタンスを考えてみましょう：

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


Since the number of processors must be integral, we conclude that $m_{N}\gets 3$. The corresponding value for $\mathcal{S}_{N}$ is equal to



プロセッサ数は整数である必要があるため、$m_{N}\gets 3$と結論します。対応する$\mathcal{S}_{N}$の値は次のようになります：

$(\frac{\text{work}_{N}-\text{span}_{N}}{m_{N}}+\text{span}_{N})= (\frac{120-40}{3}+40)=26\frac{2}{3}+40=66\frac{2}{3}$


**Algorithm 1** Computing values for $m_{N},\mathcal{S}_{N}$.
--
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
```
Input:$(\langle\text{work}_{O},\text{span}_{O},\text{work}_{N},\text{span}_{N},D \rangle,m)$ Output:failure, or values for $m_{N}$, $\mathcal{S}_{N}$
1begin
2if$(m<\lceil(\text{work}_{O}-\text{span}_{O})/(D-\text{span}_{O})\rceil )$then
3return$(\text{failure})$ /* The test of Inequality 1 cannot guarantee that the deadline will be met on $m$ processors */
4 end if
5$A\leftarrow\text{span}_{N}$
6$B\gets m\times(D-(\text{span}_{O}+\text{span}_{N}))-(\text{work}_{O}- \text{span}_{O})+(\text{work}_{N}-\text{span}_{N})$
7$C\leftarrow(-1)\times m\times(\text{work}_{N}-\text{span}_{N})$
8$m_{N}\leftarrow\big{\lceil}(-1\times B+(\sqrt{B^{2}-4\times A\times C} )/(2\times A))\big{\rceil}$
9$\mathcal{S}_{N}\leftarrow\text{span}_{N}+(\text{work}_{N}-\text{span}_{N})/m_{N}$
10 return$(m_{N},\mathcal{S}_{N})$
11 end while
```
----


Pseudo-code representation.It may be verified that Expression 8 can be rewritten to be of the form

$A\times m_{N}^{2}+B\times m_{N}+C\geq 0$

with $A,B$, and $C$ assigned the following values:

$A \leftarrow \text{span}_{N}$ $B \leftarrow (m(D-(\text{span}_{O}+\text{span}_{N}))-(\text {work}_{O}-\text{span}_{O})+(\text{work}_{N}-\text{span}_{N}))$ $C \leftarrow -1\times m\times(\text{work}_{N}-\text{span}_{N})$

The pseudo-code in Algorithm listing 1 finds the positive root of this quadratic inequality; the ceiling of which denotes the number $m_{N}$ of processors needed - this computation occurs in Line 8. In Line 9 the value computed for $m_{N}$ is used to determine the value to be assigned to $\mathcal{S}_{N}$.

Run-time complexity.Algorithm 1 comprises straight-line code with no loops or recursive calls. Hence given as input the parameters specifying a task, it is evident that Algorithm 1 has constant - $\Theta(1)$ - run-time.

擬似コードの表現。式8を以下の形式に書き換えることができることを確認できます：

$$A\times m_{N}^{2}+B\times m_{N}+C\geq 0$$

ここで、$A,B,C$は次の値が割り当てられています：

$$A \leftarrow \text{span}_{N}$$
$$B \leftarrow (m(D-(\text{span}_{O}+\text{span}_{N}))-(\text {work}_{O}-\text{span}_{O})+(\text{work}_{N}-\text{span}_{N}))$$
$$C \leftarrow -1\times m\times(\text{work}_{N}-\text{span}_{N})$$

アルゴリズムリスト1の擬似コードは、この二次不等式の正の根を見つけます。天井関数を使って必要なプロセッサの数$m_{N}$を示し、この計算はLine 8で行われます。Line 9では、計算された$m_{N}$の値を使用して$\mathcal{S}_{N}$に割り当てる値を決定します。

実行時間の複雑さ。Algorithm 1はループや再帰呼び出しがない直線的なコードから成り立っています。したがって、タスクを指定するパラメータを入力として与えられた場合、Algorithm 1の実行時間は定数であることが明らかであり、$\Theta(1)$です。


## 5 Achieving Greater Efficiency: A More Aggressive Approach

In an attempt to achieve efficiency (reducing the number of processors used in the "common case") while maintaining correctness (guaranteeing to meet deadlines provided task behavior does not exceed the worst-case bounds of $\text{work}_{O}$ and $\text{span}_{O}$), the approach derived in Section 4.2 above uses the nominal parameter values $\text{work}_{N}$ and $\text{span}_{N}$ to assign values to $m_{N}$ and $\mathcal{S}_{N}$. In this section, we propose a more aggressive approach to achieving perhaps greater efficiency without compromising correctness in any manner. This more aggressive approach is based upon exploiting insights regarding(i) the probabilistic characterization of the run-time behavior of the system; and (ii) the typical behavior of List Scheduling.

「一般的な場合」で使用されるプロセッサの数を減らしながら（効率を向上させる試み）、同時に正確性を維持することを目指します（タスクの振る舞いが$\text{work}{O}$および$\text{span}{O}$の最悪の場合を超えない限り、締め切りを満たすことを保証します）。前述のセクション4.2で導出されたアプローチでは、名目のパラメータ値$\text{work}{N}$および$\text{span}{N}$を使用して$m_{N}$および$\mathcal{S}_{N}$に値を割り当てます。このセクションでは、正確性を損なうことなく、おそらくより高い効率を達成するためのより攻撃的なアプローチを提案します。このより攻撃的なアプローチは、（i）システムの実行時の振る舞いの確率的な特性に関する洞察、および（ii）List Schedulingの典型的な振る舞いに関する洞察を活用することに基づいています。

The probabilistic characterization of run-time behavior.As discussed in Section 2 (and illustrated in Examples 1 and 2), we may have a _probability_ associated with the likelihood that the $\text{work}_{N}$ and $\text{span}_{N}$ parameter values are correct. We may, for example be able to assert that there is a $\leq 0.05$ probability that the actual work will exceed $\text{work}_{N}$, and a $\leq 0.05$ probability that the actual span will exceed $\text{span}_{N}$. Now this threshold probability of 0.05 may have been selected because we desire that the probability that the $(m-m_{N})$ sleeping processors will need to be awakened be $\leq 0.05$. If so, the method for computing $\mathcal{S}_{N}$ and $m_{N}$ described in Section 4.2 above may be overly conservative since the work and span distributions may not be perfectly correlated - if they are not, the probability that both the work would exceed $\text{work}_{N}$**and** the span exceed $\text{span}_{N}$, during a particular execution of the task is smaller than 0.05. (In the extreme if the two distributions are more or less independent, the probability is closer to $0.05^{2}$ which equals $.0025$, a value that is is far smaller than the sought-for threshold probability of 0.05.)

実行時の振る舞いの確率的特性。セクション2で議論されたように（Examples 1および2で示されている通り）、$\text{work}_{N}$および$\text{span}_{N}$のパラメータ値が正しい確率に関連付けられることがあります。たとえば、実際の作業が$\text{work}_{N}$を超える確率が$\leq 0.05$であると主張できるかもしれませんし、実際のスパンが$\text{span}_{N}$を超える確率も$\leq 0.05$であると言えるかもしれません。この0.05という閾値確率は、$(m-m_{N})$のスリーププロセッサが起動する必要がある確率が$\leq 0.05$であることを希望して選択されたかもしれません。もしそうなら、セクション4.2で説明されている$\mathcal{S}_{N}$と$m_{N}$を計算する方法は、作業とスパンの分布が完全に相関していない場合、過度に保守的かもしれません。もし相関していない場合、特定のタスクの実行中に作業が$\text{work}_{N}$を超える**かつ**スパンが$\text{span}_{N}$を超える確率は0.05よりも小さくなります。（極端な場合、2つの分布がほぼ独立している場合、確率は$0.05^{2}$に近くなり、これは0.05の閾値確率よりもはるかに小さい値です。）



Some observations on List Scheduling.Assuming that the actual work and span parameters of the computation do not exceed $\text{work}_{N}$ and $\text{span}_{N}$ respectively, in Section 4.2 we used Expression 1 to assign values to $m_{N}$ and $\mathcal{S}_{N}$ in a manner guaranteeing that the computation will complete execution within an interval of duration $\mathcal{S}_{N}$ upon $m_{N}$ processors. Note that Expression 1 is an _upper bound_ on the makespan of a List Scheduling generated schedule of a DAG; this upper bound is tight only for DAGs possessing a very specific structural form and/ or List Scheduling making a particular sequence of scheduling decisions (and then only if each node of the DAG executes for its entire WCET). Simulation experiments using randomly-generated graphs seem to indicate that these structures and scheduling decisions are relatively rare; for randomly-generated graphs, the makespans of actual list-scheduling generated schedules tend to cluster closer towards the lower end of the interval between the upper bound of Expression 1 and the obvious lower bound of

$\max(\frac{\text{work}}{m},\text{span})$

even if each node of the DAG does actually execute for its entire WCET. To illustrate this, we randomly generated 1000-node DAGs with varying numbers of edges in the manner described in Section 5.1 below; for each, we computed the lower bound of Expression 9, the actual makespan using a list scheduling implementation, and the upper bound of Expression 1, for scheduling the DAG upon a 10-processor platform. The results are listed in Table 1. The right-most column - the one titled "Ratio" - denotes the fraction of the interval between the lower bound and the upper bound upon which the actual makespan encroaches.

以下は、与えられた文章の日本語訳です。

セクション4.2で、計算の実際の作業とスパンのパラメータがそれぞれ$\text{work}_{N}$と$\text{span}_{N}$を超えないと仮定して、式1を使用して$m_{N}$と$\mathcal{S}_{N}$に値を割り当てました。これにより、$m_{N}$プロセッサ上での計算が$\mathcal{S}_{N}$の間隔内で完了することが保証されます。式1はDAGのリストスケジューリング生成スケジュールのmakespanの上限であり、この上限はDAGが非常に特定の構造的形状を持っている場合、またはリストスケジューリングが特定のスケジューリング決定の順序を取っている場合にのみ厳密です（そして、DAGの各ノードがその全体のWCETで実行される場合のみ）。ランダムに生成されたグラフを使用したシミュレーション実験では、これらの構造やスケジューリングの決定は比較的まれであることが示されています。ランダムに生成されたグラフの場合、実際のリストスケジューリング生成スケジュールのmakespanは、式1の上限と次の明らかな下限の間の間隔の下側に密集しています。

$\max(\frac{\text{work}}{m},\text{span})$

この例を示すために、セクション5.1で説明されている方法でさまざまなエッジの数を持つ1000ノードのDAGをランダムに生成しました。各DAGについて、10プロセッサプラットフォーム上でのDAGのスケジューリングのための式9の下限、リストスケジューリング実装を使用した実際のmakespan、および式1の上限を計算しました。結果は表1にリストされています。最も右の列 - "Ratio"というタイトルの列 - は、実際のmakespanが下限と上限の間の間隔にどれだけ進入するかの割合を示しています。

More aggressive computation of $m_{N}$ and $\mathcal{S}_{N}$.We highlight the fact that being too optimistic in assigning values to $\mathcal{S}_{N}$ and $m_{N}$ does _not_ compromise correctness: the sole effect is upon efficiency in terms of the number of processors we are able to maintain in sleep mode, and the likelihood that these processors will need to be switched on during some run of the system. Hence one possible -more aggressive- approach towards achieving greater resource efficiency during run-time would be to assign $\mathcal{S}_{N}$ a value between the lower and upper bounds of Expressions 9 and 1 as follows (rather than according to Expression 7):

$\mathcal{S}_{N}\leftarrow\max(\frac{\text{work}_{N}}{m_{N}},\text{span }_{N})+\alpha\cdot[(\frac{\text{work}_{N}-\text{span}_{N }}{m}+\text{span}_{N})-\max(\frac{\text{work}_{N}}{m_{N}},\text{ span}_{N})] $

with $\alpha,0\leq\alpha\leq 1$ a "tuning" parameter: the smaller the value of $\alpha$, the more aggressive the choice of $\mathcal{S}_{N}$. (An intuitive interpretation of the tradeoff here is that the smaller the value of $\alpha$, the greater the number of processors we can switch off, but the greater the likelihood that they will need to be awakened during some execution of the task.) For the randomly-generated DAGs of Table 1, a value of $\alpha\geq 0.208$ would have been safe: during run-time the sleeping processors are not awakened as long as the task's run-time behavior does not violate its nominal parameters $\rm{work}_{N}$ and $\rm{span}_{N}$.

$\mathcal{S}{N}$と$m_{N}$への値の割り当てに過度に楽観的であっても、正確性は_損なわれません_。影響は効率にのみ関係し、それは我々がスリープモードで保持できるプロセッサの数や、これらのプロセッサがシステムの実行中にオンにする必要がある可能性に関係しています。したがって、ランタイム中のリソース効率を向上させるための一つの可能な - より積極的な - アプローチは、式7に従うのではなく、式9と1の下限と上限の間の値に$\mathcal{S}_{N}$を割り当てることです：

$\mathcal{S}{N}\leftarrow\max(\frac{\text{work}{N}}{m_{N}},\text{span }{N})+\alpha\cdot[(\frac{\text{work}{N}-\text{span}{N }}{m}+\text{span}{N})-\max(\frac{\text{work}{N}}{m{N}},\text{ span}_{N})] $

ここで、$\alpha$は0以上1以下の「チューニング」パラメータです。$\alpha$の値が小さいほど、$\mathcal{S}{N}$の選択はより積極的になります。 (ここでのトレードオフの直感的な解釈は、$\alpha$の値が小さいほど、オフにできるプロセッサの数が増えるが、タスクの実行中にそれらを起こす必要がある可能性が高まる、ということです。)表1のランダムに生成されたDAGについて、$\alpha\geq 0.208$の値は安全でした。ランタイム中、タスクの実行時の振る舞いがその名目上のパラメータ$\rm{work}{N}$と$\rm{span}_{N}$を侵害しない限り、スリープ中のプロセッサは起こされません

If $\mathcal{S}_{N}$ is assigned a value according to Expression 10 rather than Expression 7, it is no longer the case that solving Expression 8 yields the desired value of $m_{N}$. We have not attempted to derive a closed-form solution for $m_{N}$ when Expression 10 is used in place of Expression 7; rather, we iterate through candidate values for $m_{N}$ over the range $[1,m)$, stopping at the first such value for which this value for $m_{N}$, and the resulting value for $\mathcal{S}_{N}$ computed according to Expression 10, causes Condition 6 to evaluate to true. This more aggressive approach to computing $m_{N}$ and $\mathcal{S}_{N}$ therefore has run-time complexity $\Theta(m)$ where $m$ denotes the number of processors available; a straightforward application of the idea of binary search reduces this to $\Theta(\log m)$.

式7ではなく、式10に従って$\mathcal{S}{N}$に値を割り当てると、式8を解くことで$m{N}$の所望の値が得られるわけではなくなります。私たちは、式7の代わりに式10を使用した場合の$m_{N}$の閉じた形の解を導出しようとはしていません。むしろ、$[1, m)$の範囲で$m_{N}$の候補の値を順番に試し、この$m_{N}$の値と、式10に従って計算される$\mathcal{S}{N}$の結果の値が、条件6を真に評価する最初の値で停止します。したがって、$m{N}$と$\mathcal{S}_{N}$を計算するこのより積極的な

アプローチの実行時間の複雑さは$\Theta(m)$であり、ここで$m$は利用可能なプロセッサの数を示しています。バイナリサーチのアイデアを直接適用することで、これは$\Theta(\log m)$に削減されます。


\begin{table}
\begin{tabular}{|l|c c c|c|} \hline  & \multicolumn{3}{c|}{**M A K E S P A N**} & \\
**\# edges** & **Lower (Exp. 9)** & **Actual** & **Upper (Exp. 1)** & **Ratio** \\ \hline
977 & 2627 & 2667 & 2818 & 0.208 \\
2017 & 2539 & 2587 & 2889 & 0.137 \\
4921 & 2567 & 2603 & 3222 & 0.055 \\
9935 & 2554 & 2709 & 3725 & 0.132 \\
20094 & 2599 & 2977 & 4774 & 0.174 \\
39935 & 4056 & 4113 & 6154 & 0.027 \\
50036 & 4454 & 4480 & 6491 & 0.013 \\
60212 & 5674 & 5674 & 7658 & 0.000 \\ \hline \end{tabular}
\end{table}


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



### The Experiments Reported in Table 1

We now briefly describe the experimental procedure used to generate the data populating Table 1. Graphs were synthesized using a DAG-generating variant of the well-known Erdos-Renyi method [26] for generating random graphs. The Erdos-Renyi method, given parameters $(n,p)$, yields a graph on $n$ vertices in which each edge has an independent probability $p$ of existence. We modified this method to generated _directed acyclic_ graphs with a target number of edges. Specifically,

* The number of vertices in the DAG, $n$, the maximum WCET parameter for a vertex $w$, and the desired number of edges $e$, are specified. The number of processors $m$ upon which the DAG is to be scheduled is also specified.
* Each vertex is assigned a WCET parameter that is a randomly and uniformly drawn integer over the range $(1,w)$.
* The parameter $p$, denoting the probability of existence of each edge, is computed as follows: $p\leftarrow\frac{2e}{n\times(n-1)}$ The idea is that since a DAG with $n$ vertices has a maximum of $n/\times(n-1)/2$ edges, if we were to create each such edge independently with probability $p$, on average the desired number $e$ of edges would be created.
* All edges are assumed to be directed from the lower-indexed vertex to the higher-indexed vertex (hence a topological sorting of the DAG could yield the vertices in order of increasing index). Each such edge is created with probability $p$, as follows: for i := 1 to n  for j := (i+1) to n  create edge (i,j) with probability p
* The work and span parameter of the generated DAG are computed, assuming that each vertex executes for exactly its WCET parameter value (i.e., WCET parameters are taken to represent the actual execution duration, rather than an upper bound on the execution duration). Using these values, a lower bound on makespan as given by Expression 9, and an upper bound as given by Expression 1, for the DAG upon the specified number $m$ of processors are computed.
* A schedule of the DAG upon $m$ processors using a standard implementation of list scheduling is generated (once again assuming that each vertex executes for exactly its WCET parameter value). The makespan of the resulting schedule is recorded.
* Each data-point reported in Table 1 was obtained by generating one hundred such graphs, computing the reported parameters upon each, and taking their averages.

以下は、与えられた文章の日本語訳です。

Table 1にデータを入力するための実験手順を簡単に説明します。グラフは、ランダムなグラフを生成するためのよく知られたErdos-Renyi法 [26] のDAG（有向非巡回グラフ）生成バリアントを使用して合成されました。Erdos-Renyi法は、パラメータ$(n,p)$が与えられると、各辺が存在する独立確率$p$を持つ$n$頂点のグラフを生成します。私たちはこの方法を修正して、目標となる辺の数を持つ_有向非巡回_グラフを生成しました。具体的には、

* DAGの頂点数$n$、頂点の最大WCETパラメータ$w$、および所望の辺の数$e$が指定されます。DAGがスケジュールされるプロセッサの数$m$も指定されます。
* 各頂点には、範囲$(1,w)$の間でランダムかつ一様に選ばれた整数としてWCETパラメータが割り当てられます。
* パラメータ$p$は、各辺の存在確率を示しており、次のように計算されます: $p \leftarrow \frac{2e}{n \times (n-1)}$。考え方としては、$n$の頂点を持つDAGは、最大で$n \times (n-1) / 2$の辺を持つので、各辺を独立して確率$p$で作成すると、平均して所望の辺数$e$が作成されるはずです。
* すべての辺は、インデックスの低い頂点から高い頂点への向きを持つと仮定されます（したがって、DAGのトポロジカルソートは、インデックスの増加順に頂点を生成できます）。各辺は、次のようにして確率$p$で作成されます: i := 1からnまで、j := (i+1)からnまで、確率pで辺(i,j)を作成します。
* 生成されたDAGのworkパラメータとspanパラメータは計算され、各頂点が正確にそのWCETパラメータ値で実行されると仮定しています（つまり、WCETパラメータは実際の実行期間を示すものとして、実行期間の上限としてではなく取られます）。これらの値を使用して、指定されたプロセッサ数$m$のDAGに対するExpression 9によるmakespanの下限と、Expression 1による上限が計算されます。
* 標準的なリストスケジューリングの実装を使用して、$m$プロセッサ上のDAGのスケジュールが生成されます（再び、各頂点が正確にそのWCETパラメータ値で実行されると仮定しています）。結果として得られるスケジュールのmakespanが記録されます。
* 表1に報告されている各データポイントは、100のこのようなグラフを生成し、それぞれの上で報告されたパラメータを計算し、その平均を取ることで得られました。

## 6 Summary and Conclusions

Although DAG-based models for representing parallelizable real-time code have proved very popular in the real-time scheduling theory community, they suffer from several shortcomings that restrict their usefulness in representing some kinds of real-time code. In this paper, we have explored an alternative model, one that is based upon characterizing a task by just two parameters - work and span - with two estimates on upper bounds on the value of each parameter - one that may be very large but is trust-worthy to a very high level of assurance, and a second that is smaller and is more reflective of typical or nominal behavior. We have developed an algorithm for scheduling tasks that are so modeled upon a dedicated cluster of processors in a manner guaranteeing _correctness_ - deadlines are always met provided run-time behavior does not violate the high-assurance bounds - while striving for _efficiency_ - many processors can remain in sleep mode much of the time, only being switched on in rare circumstances when run-time behavior exceeds the normal bounds.

実時間スケジューリング理論のコミュニティで非常に人気があるDAGベースのモデルを使用して、並列可能なリアルタイムコードを表現する方法が広まってきましたが、いくつかの短所があり、特定の種類のリアルタイムコードを表現するうえでその有用性を制限しています。本論文では、代替モデルを探求しています。このモデルは、タスクをわずか2つのパラメーター、すなわちworkとspanで特徴付けるものであり、各パラメータの値の上限に対して2つの推定値を持っています。一つは非常に大きいかもしれませんが、非常に高い確度で信頼性があり、もう一つは典型的または名目上の振る舞いをより反映した小さな値です。私たちは、このようにモデル化されたタスクを、専用のプロセッサクラスタ上でスケジューリングするためのアルゴリズムを開発しました。このアルゴリズムは、_正確性_を保証する方法で動作します。つまり、実行時の動作が高信頼性の境界を超えない限り、デッドラインは常に満たされます。一方、_効率性_を追求しています。多くのプロセッサはほとんどの時間、スリープモードのままであり、実行時の動作が通常の境界を超える稀な状況でのみオンに切り替えられます。

The model for representing parallelizable code that is being proposed in this paper, and the associated run-time scheduling algorithm, is particularly suitable for a certain kind of real-time application: one that repeatedly (i.e., periodically or sporadically) monitors the external environment seeking to detect some particular kind of anomalous sensory input. Most of the time the sought-for input is not detected, and not much computation needs to be performed. But on the rare occasions when the anomalous input is detected, considerable additional processing of such input is necessary; furthermore, such processing is highly parallel in nature. Example applications of this kind include real-time intrusion detection,vision-based monitoring systems, etc. For such systems, we would expect that the nominal workload, as represented by the work${}_{N}$ and span${}_{N}$ parameters, is quite small and may not exhibit much parallelism; however the workload upon "overload" (i.e., when the monitored-for condition occurs) is quite intensive (work${}_{O}$ is large) but exhibits considerable parallelism (i.e., span${}_{O}$ is relatively small compared to work${}_{O}$: equivalently, the ratio work${}_{O}$/span${}_{O}$ is large). If the system is hard-real-time, resource allocation for guaranteeing correctness must be made under worst-case assumptions - the work${}_{O}$ and span${}_{O}$ parameters. However, much of these allocated resources will remain unused much of the time during run-time; by being able to determine in a timely manner precisely when these unused resources will be needed during run-time. our approach allows us to place these resources in sleep mode until needed.

本論文で提案されている並列化可能なコードを表現するモデル、および関連する実行時スケジューリングアルゴリズムは、特定の種類のリアルタイムアプリケーションに特に適しています。すなわち、外部環境を繰り返し（周期的にまたは散発的に）監視して、特定の種類の異常な感覚入力を検出しようとするアプリケーションです。ほとんどの場合、求められている入力は検出されず、それほど多くの計算は必要ありません。しかし、異常な入力が検出される稀な機会には、そのような入力のかなりの追加処理が必要です。さらに、その処理は高度に並列的な性質を持っています。この種の例としては、リアルタイムの侵入検出、ビジョンベースの監視システムなどがあります。このようなシステムでは、work${}{N}$ と span${}{N}$ パラメータで表される名目上のワークロードはかなり小さく、あまり並列性を持っていないことが予想されます。しかし、監視条件が発生した場合の「オーバーロード」時のワークロードはかなり集中的で、work${}{O}$ は大きいですが、かなりの並列性を示しています（すなわち、span${}{O}$ は work${}{O}$ に比べて相対的に小さい。同等に、work${}{O}$/span${}{O}$ の比率は大きい）。システムがハードリアルタイムである場合、正確性を保証するためのリソースの割り当ては、最悪の場合の仮定の下で行われる必要があります - work${}{O}$ と span${}_{O}$ パラメータを使用して。しかし、これらの割り当てられたリソースの多くは、実行時のほとんどの時間で未使用のままになります。私たちのアプローチにより、これらの未使用のリソースが実行時に正確にいつ必要になるかを適切に判断することで、これらのリソースを必要なときまでスリープモードにしておくことができます。


We believe our main contribution here is the model for parallel tasks - the run-time scheduler is presented as proof-of-concept evidence of the potential benefits, in terms of resource-efficiency, of adopting this model. As future work we plan to demonstrate the model's applicability in a wider range of settings: under different scheduling paradigms (such as global EDF and global Fixed-Priority). We are also working on further extending the task model if additional profiling data of the task's run-time behavior is available (and known to be reliable). For example, straight-forward generalizations allow us to specify multiple sets of parameter values at different probability thresholds (rather than just two sets of values) - are we able to develop scheduling strategies that can meaningfully exploit such additional information?

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