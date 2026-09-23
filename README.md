# CNOT
* quantum-compute

2026.09.23

* https://arxiv.org/abs/2607.28598v1
  * https://claude.ai/chat/a07cd8c7-106e-45a6-8502-470e412d84ff

* https://arxiv.org/abs/2607.22248
  * https://claude.ai/chat/ab979d53-5a8b-44bd-8d81-d9f67b63266f

------------
* https://claude.ai/chat/eb0839c4-996e-491e-b91d-5cee3ed44fbd

# Q  Help to visualize as html/JS single file app:
* $$d(M,I)\le40.$$ Теперь если другая ветвь поиска уже потратила 41 ход, ее можно сразу выбросить. Это очень полезно для branch-and-bound: PMH дает конструктивную верхнюю границу, потому что не просто оценивает расстояние, а реально строит путь до $I$. Но здесь есть важная поправка к исходному тексту: фраза «гауссова длина текущего состояния — оценка сколько ещё идти» может ввести в заблуждение. Это не admissible heuristic в стиле A*, то есть не lower bound на оставшееся расстояние. Это upper bound: $$d^*(M,I)\le d_{\rm PMH}(M,I).$$ Для pruning это полезно, но используется иначе, чем lower-bound heuristic. 5. Еще интереснее — PMH как teacher Можно генерировать миллионы случайных $M$ и для каждой быстро получить: $$M\rightarrow M_1\rightarrow M_2\rightarrow\cdots\rightarrow I$$ с помощью PMH. Получается бесплатный dataset: $$(M,\text{best PMH action}),(M_1,\text{best PMH action}),\dots$$ На нем можно предварительно обучить policy network: $$\pi_\theta(a\mid M).$$ А потом уже улучшать ее поиском. Это стандартная идея: expert → imitation learning → search → better trajectories → retraining. И тут PMH становится не конкурентом CayleyPy, а учителем для CayleyPy. 6. А вот последнее утверждение я бы воспринимал осторожно В тексте написано: «диффдист можно учить … регрессией на гауссову длину $M$» Это можно делать, но тогда сеть учится предсказывать $$d_{\rm PMH}(M),$$ а не настоящее расстояние в графе Кэли $$d^*(M,I).$$ Это принципиально разные targets. Если цель CayleyPy — научиться приближать истинный shortest-path distance, random walks/BFS/exact solutions для малых $n$ дают более содержательную информацию. PMH labels дешевые и масштабируемые, но они несут bias самого PMH. ⸻ Самая интересная для вашего CayleyPy мысль в сообщении, на мой взгляд, вот какая: $$\boxed{\text{PMH baseline}+\text{CayleyPy search}+\text{learned heuristic}}$$ вместо $$\boxed{\text{CayleyPy против PMH}}.$$ Причем возникает очень хороший эксперимент: взять, например, случайные матрицы $GL(n,2)$ и сравнить Gaussian / PMH / CayleyPy по CNOT count и времени. А еще интереснее обучать модель не предсказывать абсолютный $d(M,I)$, а предсказывать, какой следующий CNOT позволит улучшить PMH-решение. Это превращает очень сильный классический алгоритм в стартовую точку нейропоиска.

# A

Looks good on both desktop and phone. Publishing it.The explorer is a single HTML file with three tabs, one for each main idea in the text.

**1. Synthesize one matrix** (paths from $M$ to $I$ in the Cayley graph)
- Pick $n \in [2,16]$ and a seed. The app computes a circuit four ways: Gauss–Jordan, PMH (section size $m$ set automatically or by hand), beam search with PMH rollouts, and exact BFS.
- Exact BFS runs instantly for $n\le 4$. For $n=5$ there's a button; it takes about 3 s to build all 9,999,360 distances.
- You can step through the path. Each row operation $R_t \leftarrow R_t \oplus R_c$ is highlighted on the matrix, with the target row in amber and the control row in blue.
- The circuit for $y=Mx$ is drawn alongside and fills in from the output end, since $M = E_1\cdots E_s\,M_s$.
- The **Bounds along the path** chart covers section 4 of the text. It plots three curves:
  - $t + d_{\rm PMH}(M_t)$, a constructive upper bound.
  - $t + \#\{i : R_i \ne e_i\}$, an admissible lower bound (each CNOT rewrites only one row).
  - $t + d^*(M_t)$, the exact value when available. A step where this curve rises is a step that left the shortest path.
- The beam search is branch-and-bound in the sense of the text. PMH/Gauss rollouts supply the incumbent, and any child with $t+\mathrm{LB}\ge$ best is pruned.

**2. Scaling with n**
- Average CNOT counts for Gauss and PMH (with $m$ set automatically, and with the best $m$), for $n$ up to 128.
- These are compared with the counting bound $\log_2|GL(n,2)|/\log_2 n(n-1)$.
- In the "÷ $n^2$" view, Gauss stays flat near $1/2$, while PMH and the counting bound fall roughly like $1/\log n$.

**3. PMH as a teacher** (sections 5–6)
- For $n=3,4$ it checks every element of $GL(n,2)$; for $n=5$ it samples 6000 matrices. A heatmap shows the label length against the true $d^*$.
- For example, for $n=4$ with $m=1$, PMH is optimal on only about 17% of matrices, with a mean excess of about 2 CNOTs. That makes the difference between $d_{\rm PMH}$ and $d^*$ as a regression target concrete.
- A table shows a single PMH trajectory as dataset rows $(M_t, a_t)$, next to the PMH-to-go length, a fresh PMH length from $M_t$, and the true $d^*(M_t)$.

In testing, every Gauss and PMH circuit was checked by simulation to reproduce $M$ for $n$ up to 70. Beam search matched the exact optimum on all 30 random $4\times4$ test cases.

# APPLICATION
```
/Users/paulpaul/Library/Mobile Documents/com~apple~CloudDocs/code/cnot
(base)  18:16 paulpaul ~/Library/Mobile Documents/com~apple~CloudDocs/code/cnot $  ls -ltra
total 80
-rw-r--r--@  1 paulpaul  staff  39501 Sep 23 18:14 cnot-synthesis-explorer.html
drwxr-xr-x@ 40 paulpaul  staff   1280 Sep 23 18:15 ..
drwxr-xr-x@  3 paulpaul  staff     96 Sep 23 18:15 .
(base)  18:16 paulpaul ~/Library/Mobile Documents/com~apple~CloudDocs/code/cnot $  

```






```

…or create a new repository on the command line

echo "# quantum-compute" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/fkhafizov/quantum-compute.git
git push -u origin main

…or push an existing repository from the command line

git remote add origin https://github.com/fkhafizov/quantum-compute.git
git branch -M main
git push -u origin main

```
