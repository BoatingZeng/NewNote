手册: https://mirrors.ustc.edu.cn/CTAN/info/lshort/chinese/lshort-zh-cn.pdf

下面的命题只是作为latex例子,不保证正确性。

$$
E = mc^2
$$

$$\begin{split}
A \Rightarrow B \\
\urcorner A \Leftarrow \urcorner B
\end{split}$$

$证明:度量空间X的子集M在X中稀疏 \Leftrightarrow(\overline{M})^c在X中稠密$

$在欧氏空间中,以向量长度|x|为向量x的范数。方阵作为算子,则有范数不为1的幂等算子T:$

$$
T=
\begin{bmatrix}
3 & -6 \\
1 & -2
\end{bmatrix}
$$

$$
\Vert T\Vert =\sup_{\substack{x\in\mathcal{D}(T) \\ 
\|x\|=1}}\Vert Tx\Vert =5\sqrt{2}
$$

$$
\Vert Tx\Vert 在x=\begin{bmatrix}
\sqrt{5}/5 \\ 
-2\sqrt{5}/5
\end{bmatrix}时取到最大值
$$

$T是复希尔伯特空间H上的正有界自伴线性算子,证明:\Vert T^{1/2}\Vert =\Vert T\Vert^{1/2} $

$(注意:不清楚该等式是否正确,此处只是latex例子)A是自伴算子,证明其范数\Vert A\Vert=\sup_{\|x\|=1} |\langle Ax,x\rangle |$

$X,Y为拓扑空间E的子集,给X,Y,X \cup Y以诱导拓扑,A为X \cup Y的子集,A \cap X与A\cap Y分别为X与Y的开集。A为X \cup Y的开集吗？$

$从E^{n+1}-\{0\}出发,两个点将被粘合在一起,当且仅当它们位于同一条过原点的直线上,所得的粘合空间记为Y,证明Y是Hausdorff空间。$

$从E^{n+1}-\{0\}出发,两个点将被粘合在一起,当且仅当它们位于同一条过原点的直线上且到原点距离相等,所得得空间是粘合空间吗？(AI说这个空间其实也是实射影空间)$

$E^{n+1}-\{0\}可以缩放到S^n是什么意思?(同伦)$

$G为拓扑群,H为G的子群,H在G内的左陪集组成轨道空间A,H在G内的右陪集组成轨道空间B。A与B同胚吗？$

下面3个问题还没得到解答。

$证明：\{ m+n\sqrt{2} |n,m \in Z \}在R中稠密。$

$k_1,k_2,N为正整数,N+1\ge k_1>k_2,a为某无理数,|(k_1a \space mod \space 1) - (k_2a \space mod \space 1)| < 1/N,能推出|(k_1a - k_2a \space mod \space 1| < 1/N吗？$

$a为某无理数,证明：\{ na \space mod \space 1|n \in Z\}在[0,1)中稠密。$

$若G像同胚群一样作用于单连通空间X并且每一点x \in X有邻域U使得U \cap g(U) = \varnothing 对于一切g \in G - \{e\} 成立。则\pi_1(X/G)同构于G。固定一点x_0 \in X,对于g \in G用一条道路 \gamma 把x_0连接到g(x_0)。若\pi:X \rightarrow X/G为投影,则\pi\circ\gamma是X/G内基于\pi(x_0)的一条环道。按\phi(g)=<\pi\circ\gamma>定义\phi:G\rightarrow\pi_1(X/G,\pi(x_0)),证明\phi是同态。$

$X是篦式空间,X=\bigcup_{n=1}^\infty(\{1/n\}\times[0,1])\cup(\{0\}\times[0,1])\cup([0,1]\times \{0\})。1_X是X中的恒等映射。p=(0,1)是X中一点,c_p是X中的常值映射,对一切x\in X,c_p(x)=p。证明:不存在从1_X到c_p相对于\{p\}的伦移。$

$在E^3内,A=\{(x,y,z)|z>0\},B是E^2内的道路连通空间,C=\{(x,y,z)|(x,y)\in B, -1<z<=0\}, D=A \cup C,证明D是单连通。$

$设n>1，A为E^n的紧致子集。证明E^n-A恰好有一个无界的连通分支。$