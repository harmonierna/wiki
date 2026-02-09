# Continuous Double Auction

Underpins many financial exchanges such as NASDAQ, NYSE.

- Buyers continuously submit bids (offers to buy)
- Sellers continuously submit asks (offers to sell)

_Continuous_ because orders can arrive at any time, and _double_ because both sides actively place prices.

Leads to rapid convergence on theoretical equilibrium price

# Origin

- Smith in 1962 demonstrated that a small group of humans, in a lab setting CDA, successfully converged on the equilibrium[^1].
- Gode and Sunders in 1993 explored how much intelligence is required of traders to converge on theoretical equilibrium[^2].

# Zero Intelligence

Gode and Sunders simulated a CDA populated by two automated traders:

- zero-intelligence unconstrained (ZIU)
- zero-intelligence constrained (ZIC)

Both of which generated random quote prices, with ZIC constrained to never trade at a loss.

Their paper demonstrated that these simple non-adaptive algorithms performed similarly to human traders, and concluded that intelligence has little effect on efficiency in a CDA, proposing that Smith's invisible hand may have more agency in financial markets than expected.

# Minimal Intelligence

In 97, Cliff and Bruten extended Gode and Sunder's work and proved that zero intelligence is not enough[^3].

Market conditions were found under which ZIC would fail to equilibrate, and thus proposed _zero-intelligence plus (ZIP)_.

ZIP is known as a minimal-intelligence traders, due to relying on a basic stochastic machine learning algorithm for optimisation.

Another notable minimal-intelligence trader is Gjerstad and Dickhaut's 98 model "GD"[^4] which is a probabilistic model.

# Minimal Intelligence > Human Intelligence

In 2001, IBM demonstrated that ZIP and GD repeatedly outperformed human traders in a market populated by both[^5].

Their proposal: efficient trading algorithms had business potential worth billions of dollars annually.

Many refer to their study as initiating the rise of algorithmic trading in real financial markets, which since then spawned numerous new algorithmic traders. See [^6][^7][^8][^9][^10] for detailed explanations of some of these.

# Differential Evolution

Population-based optimization algorithm used to optimize complex objective functions, especially when gradients aren't available or the problem is noisy or non-smooth. Originally developed in 1995[^11] by Storn and Price. It maintains a population of candidate solutions which it evolves over generations through mutation, crossover, and selection operations.

Let the population at generation $g$ be

$$
\text{P}_g = \{x_{i,g}\}_{i=1}^{\text{NP}}
$$

where:

- $\text{NP}$ is the population size (number of candidate solutions)
- $x_{i,g} \in \mathbb{R}^D$ is the $i$-th candidate solution vector at generation $g$
- $D$ is the dimensionality of the problem

**Algorithm Parameters:**

- $\text{F} \in (0, 2)$ is the mutation (or scaling) factor, typically $\text{F} \in [0.5, 1]$
- $\text{CR} \in [0, 1]$ is the crossover probability

## Basic Operations

For each target vector $x_{i,g}$, DE creates a trial vector $u_{i,g}$ through:

**1. Mutation:** Generate a mutant vector $v_{i,g}$ by combining randomly selected population members:

$$
v_{i,g} = x_{r_1,g} + \text{F} \cdot (x_{r_2,g} - x_{r_3,g})
$$

where $r_1, r_2, r_3 \in \{1, \ldots, \text{NP}\}$ are distinct random indices, and $r_1, r_2, r_3 \neq i$.

**2. Crossover:** Create trial vector $u_{i,g}$ by mixing components:

$$
u_{j,i,g} = \begin{cases}
v_{j,i,g} & \text{if } \text{rand}(0,1) \leq \text{CR} \text{ or } j = j_{\text{rand}} \\
x_{j,i,g} & \text{otherwise}
\end{cases}
$$

where $j \in \{1, \ldots, D\}$ and $j_{\text{rand}}$ is a randomly chosen index ensuring at least one component is inherited from the mutant.

**3. Selection:** Choose the better vector for the next generation:

$$
x_{i,g+1} = \begin{cases}
u_{i,g} & \text{if } f(u_{i,g}) \leq f(x_{i,g}) \\
x_{i,g} & \text{otherwise}
\end{cases}
$$

## Variants

### DE/rand/1

The basic DE variant where candidate solutions are chosen randomly. This is the formulation shown above in the mutation step.

### Generational vs Steady-State

**Generational:** Maintains two populations simultaneously. Iterates through the first population and copies either the current solution or its mutated equivalent into the second population based on performance. At the end of iterating through the first population, the second population is complete, representing the next generation.

**Steady-State:** A single trial solution is selected at random from the population for each iteration. A mutated solution is created and evaluated against the trial solution. The better performing solution is immediately written back into the population, replacing the trial solution.

The steady-state implementation uses the same mutation formula as above, with the trial solution being $x_{r_1,g}$.

## JADE: Adaptive Differential Evolution

JADE (proposed by Zhang and Sanderson, 2009)[^12] is an adaptive variant that removes the need for trial-and-error tuning of $F$ and $\text{NP}$. It employs a greedier mutation strategy with mechanisms to prevent premature convergence.

### DE/current-to-pbest/1

The mutation formula differs from DE/rand/1:

$$
v_{i,g} = x_{i,g} + F_i \cdot (x_{\text{best}}^p - x_{i,g}) + F_i \cdot (x_{r_1,g} - x_{r_2,g})
$$

where:

- $x_{i,g}$ is the trial (current) solution
- $x_{\text{best}}^p$ is randomly chosen from the top $100p\%$ best solutions in the population
- $x_{r_1,g}, x_{r_2,g}$ are randomly chosen: $r_1 \neq r_2 \neq i$
- $F_i$ is a unique mutation factor for each trial solution
- Typically $p \in [5\%, 20\%]$

### With Archive

JADE can use an archive $A$ of replaced solutions to drive progress:

$$
v_{i,g} = x_{i,g} + F_i \cdot (x_{\text{best}}^p - x_{i,g}) + F_i \cdot (x_{r_1,g} - \tilde{x}_{r_2,g})
$$

where $\tilde{x}_{r_2,g} \in P \cup A$ (selected from population or archive). When a mutated solution replaces its trial solution, the replaced solution is added to $A$. If $|A| > \text{NP}$, solutions are randomly removed until $|A| = \text{NP}$.

### Adaptive Mutation Factor

Each mutation factor $F_i$ is generated using a Cauchy distribution:

$$
F_i = \text{randc}_i(\mu_F, 0.1)
$$

The mean $\mu_F$ is updated after each generation (or every $\text{NP}$ evaluations in steady-state):

$$
\mu_F = (1 - c) \cdot \mu_F + c \cdot \text{mean}_L(S_F)
$$

where:

- $c \in [1/20, 1/5]$ is the adaptation rate
- $S_F$ is the set of successful $F_i$ values from the generation
- $\text{mean}_L$ is the Lehmer mean:

$$
\text{mean}_L(S_F) = \frac{\sum_{F \in S_F} F^2}{\sum_{F \in S_F} F}
$$

The Lehmer mean propagates larger mutation factors to improve progress rate.

### Parameter Selection

JADE introduces two control parameters:

- $c \in [1/20, 1/5]$: adaptation rate
- $p \in [5\%, 20\%]$: greediness of mutation

**Important:** The relationship between $p$ and population size $\text{NP}$ matters. If $p = 20\%$ and $\text{NP} = 4$, then $\lceil 0.2 \times 4 \rceil = 1$, effectively implementing current-to-best rather than current-to-pbest. This degrades to a greedier, less reliable strategy. Choose $\text{NP}$ and $p$ such that multiple solutions are in the top $100p\%$ subset.

## Master's Paper Exploring JADE in Stock Exchange

![[su19941_report.pdf]]

---

[^1]: Vernon L. Smith. “An experimental study of competitive market behavior.” In “Journal of Political Economy”, p.111-137. 1962

[^2]: Dhananjay K. Gode, and Shyam Sunder. “Allocative Efficiency of Markets with Zero-Intelligence Traders: Markets as a Partial Substitute for Individual Rationality”. In “Journal of Political Economy”, vol. 101, no. 1, pp. 119-137. 1993

[^3]: Dave Cliff, and Janet Bruten. “Zero is Not Enough: On The Lower Limit of Agent Intelligence For Continuous Double Markets”. 1997.

[^4]: Steven Gjerstad, and John Dickhaut. “Price Formation in Double Auctions”. 1998.

[^5]: Rajarshi Das, James E. Hanson, Jeffrey O. Kephart, and Gerald Tesauro. “Agent-Human Interactions in the Coutinuous Double Auction”. 2001

[^6]: Gerald Tesauro, and Jonathan L. Bredin. “Strategic sequential bidding in auctions using dynamic programming”. 2002.

[^7]: Steven Gjerstad. “The impact of pace in double auction bargaining.” 2003.

[^8]: Perukrishnen Vytelingum, Dave Cliff, and Nicholas R. Jennings. “Strategic bidding in continuous double auctions”. 2008.

[^9]: Jasmina Arifovic, and John Ledyard. “A behavioral model for mechanism design: Individual evolutionary learning. In “Journal of Economic Behavior & Organization”, vol. 78, pp. 374-395.

[^10]: Dave Cliff. “Parameterised-Response Zero-Intelligence Traders”. 2022.

[^11]: Rainer Storn, and Kenneth Price. “Differential Evolution – A Simple and Efficient Heuristic for global Optimization over Continuous Spaces”. 1997

[^12]: Jingqiao Zhang, and Arthur C. Sanderson. “JADE: Adaptive Differential Evolution with Optional External Archive”. 2009.
