Proof.: Given $v_{a},v_{b}\in predecessor(v_{c})$ with $h(v_{a})\geq h(v_{b})$, we have $length(\delta_{v_{a}}\cup v_{c})>length(\delta_{v_{b}}\cup v_{c})$. Therefore, the predecessor node of $v_{\ell}$ with the latest finish is in the longest path ending with $v_{e}$ in $H(\phi_{i}^{*})$. 

#### 4.2.3 Explicit Execution Order (ESO)

Tighter boundaries can be achieved by using ESO for NC-joints, because every joint could just be interfered with by concurrent joints with higher priority. Taking the proposed scheduling, a novel analysis method is illustrated, which can sustain CPE and explicit run sequence of NC-joints.

With joint priority, the interfering joints of $v_{j}$ on $m-1$ processors could be effectively decreased to joints in N($v_{j}$) that have a higher priority than $p_{j}$, $m-1$ joints in N($v_{j}$) that have a lower priority and the highest WCET because of the non-preemptive schedule [10]. N${}^{e}(v_{j})$ are the joints that can interfere with a NC-joint $v_{j}$ with an explicit order, in Formula (3), in which $\operatorname*{argmax}_{v_{2}}^{m-1}$ returns the first $m-1$ joints with the highest value of the given metric. For the sake of simplicity, it takes $(m-1)$ low-priority joints as the upper limit. The better ILP-based method can be used to calculate this congestion accurately. In short, if joint-level preemption is allowed, N${}^{e}(v_{j})$ will further decrease to $\left\{v_{k}|q_{k}>q_{j},v_{k}\in\text{N}(v_{j})\right\}$.

\[\text{N}^{e}(v_{j})=\left\{v_{k}|q_{k}>q_{j},v_{k}\in\text{N}(v_{j})\right\} \cup\operatorname*{argmax}_{v_{k}}^{m-1}\{C_{k}|q_{k}<q_{j},v_{k}\in\text{N} (v_{j})\}\] (3)

$h(v_{j}),\forall v_{j}\in V$ could be calculated by Formula (3), N${}^{e}(v_{j})$ applied to NC-joints running on other $m-1$ processors. Therefore, $\alpha_{i}$, $\beta_{i}$ can be bounded with the updated $h(\phi_{i}^{*})$, $h(v_{j}),\forall v_{j}\in H(\phi_{i}^{*})\cup G(\phi_{i}^{*})$. Note that with an explicit schedule, $\delta_{v_{e}}$ calculated in Formula (3), it is not necessarily the longest path in $H(\phi_{i}^{*})$ that runs in the interfering workload [11]. On the contrary, $\delta_{v_{e}}$ offers the path that is always finished last because of the pre-planned joint execution sequence.

However, the final limit of the response time is different from the general situation. With joint priority, whole workload in $(C_{i}-L_{i}-\alpha_{i}-\beta_{i})$ do not have to hinder the execution of $\delta_{v_{e}}$. $R^{e}$ is the response time of the DAG task with ESO. It is defined in Formula (4), in which decides the joints that could prolong $\delta_{v_{e}}$, $l_{\delta_{v_{e}},j}$ offers the real latency on $\delta_{v_{e}}$ from joint $v_{j}$ in the interfering workload.

\[R^{e}=\beta_{i}+\sum_{\phi_{i}^{*}\in\Phi^{*}}L_{i}+\begin{cases}0&if\left| \Lambda_{\text{N}^{e}(\delta_{v_{e}})}\right|<\text{m}\\ \left|\sum_{v_{j}\in\text{N}^{e}(\delta_{v_{e}})}I_{\delta_{v_{e}},j}/m\right| &otherwise\end{cases}\] (4)

The length of $\phi_{i}^{*}(L_{i})$ and the WC latency on $\delta_{v_{e}}(I_{\delta_{v_{e}}})$ in the interfering workload, of $\phi_{i}^{*}$ is the WC completion time and $H(\phi_{i}^{*})$ is upper bounded by $\beta_{i}+L_{i}+\left[\sum_{v_{j}\in\text{N}^{e}(\delta_{v_{e}})}I_{\delta_{v_{ e}},j}/m\right|$. If the number of paths in the joints that can cause $I_{\delta_{v_{e}}}$ is smaller than $m$, $\left(\left|\Lambda_{\text{N}^{e}(\delta_{v_{e}})}\right|<m\right)$, $\delta_{v_{e}}$ runs directly after $\phi_{i}^{*}$ and finishes by $L_{i}+\beta_{i}$. Note that $I_{\delta_{v_{e}}}=0$, $\beta_{i}=0$, as whole workload in $H(\phi_{i}^{*})$ contributes to $\alpha_{i}$ so that $\phi_{i+1}^{*}$ can start immediately after $\phi_{i}^{*}$.

These joints can interfere with $\delta_{v_{e}}$ (namely, N${}^{e}(\delta_{v_{e}})$) are bound by Formula (5), in which $I_{\delta_{v_{e}},j}$ offers the real latency from joint $v_{j}$ on $\delta_{v_{e}}$.

\[\begin{array}{l}\text{N}^{e}(\delta_{v_{e}})=\underset{v_{k}\in\delta_{v_{e} }}{\cup}\{v_{j}|h(v_{j})>h(\phi_{i}^{*})\wedge q_{j}>q_{k},\forall v_{j}\in \text{N}(v_{k})\}\cup\\ \underset{v_{k}\in\delta_{v_{e}}}{\cup}\operatorname*{argmax}_{v_{k}}\{I_{ \delta_{v_{e}},j}|h(v_{j})>h(\phi_{i}^{*})\wedge q_{j}<q_{k},v_{j}\in\text{N} (v_{k})\}\end{array}\] (5)