Quantum Approximate Optimization Algorithm (QAOA)
Implementation of QAOA using Qiskit to solve a simple max-cut problem on a random graph of 12 nodes

Combinatorial optimization means searching for an optimal solution in a finite or countably infinite set of potential solutions. Optimality is defined with respect to some criterion function, which is to be minimized or maximized, which is typically called the cost function.

There are various types of optimization problems. These include Minimization: cost, distance, length of a traversal, weight, processing time, material, energy consumption, number of objects. Maximization: profit, value, output, return, yield, utility, efficiency, capacity, number of objects. Any maximization problem can be cast in terms of a minimization problem and vice versa.

The general form a combinatorial optimization problem is given by

$$ \text{maximize } \;\;      C(x)$$$$ \text{subject to } \;\; x \in S $$
where $x \in S$, is a discrete variable and $C : D \rightarrow \mathbb{R}$ is the cost function, that maps from some domain $S$ in to the real numbers $\mathbb{R}$. The variable $x$ can be subject to a set of constraints and lies within the set $S \subset D$ of feasible points.

In binary combinatorial optimization problems, the cost function $C$ can typically be expressed as a sum of terms that only involve a subset $Q \subset[n]$ of the $n$ bits in the string $x \in \{0,1\}^n$ and is written in the canonical form

$$ C(x) = \sum_{(Q,\overline{Q}) \subset [n]} w_{(Q,\overline{Q})} \; \prod_{i\in Q} x_i \; \prod_{j\in \overline{Q}} (1- x_j), $$
where $x_i \in \{0,1\}$ and $w_{(Q,\overline{Q})}\in \mathbb{R}$. We want to find the n-bit string $x$ for which $C(x)$ is the maximal.

We Consider an $12$-node non-directed graph G = (V, E) where |V| = 12 with edge weights $w_{ij}&gt;0$, $w_{ij}=w_{ji}$, for $(j,k)\in E$. A cut is defined as a partition of the original set V into two subsets. The cost function to be optimized is in this case the sum of weights of edges connecting points in the two different subsets, crossing the cut. By assigning $x_i=0$ or $x_i=1$ to each node $i$, one tries to maximize the global profit function (here and in the following summations run over indices 0,1,...n-1)

$$C(\textbf{x}) = \sum_{i,j = 1}^n  w_{ij} x_i (1-x_j).$$
where n=12;

To simplify notation, we assume uniform weights $ w_{ij} = 1$ for $(i,j) \in E$. In order to find a solution to this problem on a quantum computer, one needs first to map it to a diagonal Hamiltonian. We write the sum as a sum over edges in the set $(i,j) = E$

$$C(\textbf{x}) = \sum_{i,j = 1}^n w_{ij} x_i (1-x_j)  = \sum_{(i,j) \in E} \left( x_i (1-x_j) + x_j (1-x_i)\right)$$
To map is to a spin Hamiltonian we make the assignment $x_i\rightarrow (1-Z_i)/2$, where $Z_i$ is the Pauli Z operator that has eigenvalues $\pm 1$ and obtain $X \rightarrow H$

$$ H = \sum_{(j,k) \in E} \frac{1}{2}\left(1 - Z_j Z_k \right).$$
This means that the Hamiltonian can be written as a sum of $m = |E|$ local terms $\hat{C}_e = \frac{1}{2}\left(1 - Z_{e1}Z_{e2}\right)$ with $e = (e1,e2) \in E$.

The $MAXCUT$ problem is known to be a NP-hard problems. In fact it turns out that many combinatorial optimization problems are computationally hard to solve in general. In light of this fact, we can't expect to find a provably efficient algorithm, i.e. an algorithm with polynomial runtime in the problem size, that solves these problems. This also applies to quantum algorithms. There are two main approaches to dealing with such problems. First approach is approximation algorithms that are guaranteed to find solution of specified quality in polynomial time. The second approach are heuristic algorithms that don't have a polynomial runtime guarantee but appear to perform well on some instances of such problems.

Approximate optimization algorithms are efficient and provide a provable guarantee on how close the approximate solution is to the actual optimum of the problem. The guarantee typically comes in the form of an approximation ratio, $\gamma \leq 0$. A probabilistic approximate optimization algorithm guarantees that it produces a bit string $\textbf{x}^* \in \{0,1\}^n$ so that with high probability we have that with a positive $C_{max} = \max_{\textbf{x}}C(\textbf{x})$

$$ C_{max} \geq C(\textbf{x}^*) \geq \alpha C_{max}. $$
For the $MAXCUT$ problem there is a famous approximate algorithm due to Goemans and Williamson. This algorithm is based on an SDP relaxation of the original problem combined with a probabilistic rounding technique that yields an with high probability approximate solution $\textbf{x}^*$ that has an approximation ratio of $\alpha \approx 0.868$. This approximation ratio is actually believed to optimal so we do not expect to see an improvement by using a quantum algorithm.

The Quantum approximate optimization algorithm (QAOA) by Farhi, Goldstone and Gutmann 3 is an example of a heuristic algorithm. Unlike Goemans-Williamson algorithm, QAOA does not come with performance guarantees. QAOA takes the approach of classical approximate algorithms and looks for a quantum analogue that will likewise produce a classical bit string $x^*$ that with high probability is expected to have a good approximation ratio $\alpha$. Before discussing the details, let us first present the general idea of this approach.

We want to find a quantum state $|\psi_p(\vec{\gamma},\vec{\beta})\rangle$, that depends on some real parameters $\vec{\gamma},\vec{\beta} \in \mathbb{R}^p$, which has the property that it maximizes the expectation value with respect to the problem Hamiltonian $H$. Given this trial state we search for parameters $\vec{\gamma}^*,\vec{\beta}^*$ that maximize $F_p(\vec{\gamma},\vec{\beta}) = \langle \psi_p(\vec{\gamma},\vec{\beta})|H|\psi_p(\vec{\gamma},\vec{\beta})\rangle$.

Once we have such a state and the corresponding parameters we prepare the state $|\psi_p(\vec{\gamma}^*,\vec{\beta}^*)\rangle$ on a quantum computer and measure the state in the $Z$ basis $|x \rangle = |x_1,\ldots x_n \rangle$ to obtain a random outcome $x^*$.

We will see that this random $x^*$ is going to be a bit string that is with high probability close to the expected value $M_p = F_p(\vec{\gamma}^*,\vec{\beta}^*)$. Hence, if $M_p$ is close to $C_{max}$ so is $C(x^*)$.

The Qiskit Implementation
As the example implementation we consider the $MAXCUT$ problem on the butterfly graph of the openly available IBMQ 5-qubit chip. The graph will be defined below and corresponds to the native connectivity of the device. This allows us to implement the original version of the $QAOA$ algorithm, where the cost function $C$ and the Hamiltonian $H$ that is used to generate the state coincide. Moreover, for such a simple graph the exact cost function can be calculated analytically, avoiding the need to find optimal parameters variationally. To implement the circuit, we follow the notation and gate definitions from the Qiskit Documentation.
