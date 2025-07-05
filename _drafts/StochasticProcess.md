---
published: false
title: "Stochastic Process"
tags:
- ML
---

A **stochastic process** is a mathematical object used to describe systems that evolve over time in a probabilistic manner. For a seasoned software engineer, you can think of it as a **function over time whose values are random variables**, rather than deterministic outcomes.

### Key Concepts:

* **Index Set**: Usually time $t \in T$, which can be discrete (e.g., $T = \mathbb{N}$) or continuous (e.g., $T = \mathbb{R}^+$).
* **State Space**: The set of values the process can take, e.g., $\mathbb{R}$, $\mathbb{Z}$, or even more complex structures.
* **Sample Path**: A single realization of the process—like one simulated run. For example, a single stock price evolution over a year.
* **Random Variables**: For each time $t$, the value $X(t)$ is a random variable. So the process is a collection $\{X(t)\}_{t \in T}$.

### Example Types:

* **Discrete-Time, Discrete-State**: Markov chains.
* **Continuous-Time, Continuous-State**: Brownian motion (Wiener process).
* **Poisson Process**: Models random events occurring over time with a known average rate.

### Practical Analogy:

If you think of a typical system like server load over time, where inputs are unpredictable, a stochastic process models that load as a distribution over time—giving you tools not only to simulate but also to reason statistically about future states, correlations, and expected behaviors.

To classify a **continuous-time stochastic process** as **Brownian motion** (or a **Wiener process**), it must satisfy a very specific set of properties. These aren't just mathematical niceties—they define the behavior that makes Brownian motion so central in areas like quantitative finance, physics, and signal processing.

---

### A process $\{B(t)\}_{t \geq 0}$ is called **Brownian motion** if:

1. **Initial Condition**
   $B(0) = 0$ almost surely.

2. **Independent Increments**
   For any $0 \leq t_0 < t_1 < \dots < t_n$, the increments

   $$
   B(t_1) - B(t_0), B(t_2) - B(t_1), \dots, B(t_n) - B(t_{n-1})
   $$

   are **independent random variables**.

3. **Stationary Increments**
   The distribution of an increment depends only on the time lag:

   $$
   B(t+s) - B(s) \sim \mathcal{N}(0, t)
   $$

   for any $s, t \geq 0$. So, the increment over any interval of length $t$ is normally distributed with mean 0 and variance $t$, regardless of where the interval starts.

4. **Gaussian Increments**
   Increments are normally distributed:

   $$
   B(t) - B(s) \sim \mathcal{N}(0, t - s)
   $$

   for all $0 \leq s < t$.

5. **Continuous Paths**
   Almost all sample paths $t \mapsto B(t)$ are continuous (but not differentiable).

---

### Important Notes

* **Brownian motion is memoryless in increments**—it has no "drift" or autocorrelation.
* If you introduce a deterministic drift or scale (e.g., $\mu t + \sigma B(t)$), you get a **Brownian motion with drift** or **geometric Brownian motion**.
* It's a **Markov process** and also a **martingale**.

**Brownian motion** is just one example of a **continuous-time, continuous-state stochastic process**. There are several others, each characterized by different properties (e.g., distribution of increments, memory, variance structure). Here's a breakdown of major families:

---

### 1. **Brownian Motion (Wiener Process)**

* Zero mean, linear variance.
* Independent, normally distributed increments.
* No memory.

Used for modeling random noise, stock prices (base model), physical diffusion.

---

### 2. **Geometric Brownian Motion (GBM)**

* Defined as $X(t) = X(0) e^{(\mu - \frac{1}{2}\sigma^2)t + \sigma B(t)}$.
* Models exponential growth with volatility.

Used in **Black-Scholes** and most asset pricing.

---

### 3. **Ornstein–Uhlenbeck Process**

* Mean-reverting process.
* SDE:

  $$
  dX(t) = \theta(\mu - X(t))dt + \sigma dB(t)
  $$
* Variance is bounded over time.

Used in interest rate modeling (e.g., **Vasicek model**), noise with memory (e.g., thermodynamics).

---

### 4. **Fractional Brownian Motion (fBm)**

* Generalization of Brownian motion with **long-range dependence**.
* Hurst parameter $H \in (0,1)$ controls memory:

  * $H = 0.5$: standard Brownian motion
  * $H > 0.5$: positive correlation (persistent)
  * $H < 0.5$: negative correlation (mean-reverting)

Used in **network traffic**, **geophysics**, and **fractal modeling**.

---

### 5. **Poisson Random Measures & Lévy Processes**

* Jump processes with continuous time but not necessarily continuous paths.
* **Lévy processes** generalize Brownian motion and Poisson processes:

  * Allow for **jumps**, infinite variance, etc.
  * Includes **stable processes**, **CGMY**, **Variance Gamma**.

Used in modeling market crashes, **heavy tails**, and **non-Gaussian returns**.

---

### 6. **Cox–Ingersoll–Ross (CIR) Process**

* Mean-reverting but keeps values strictly positive:

  $$
  dX(t) = \theta(\mu - X(t))dt + \sigma \sqrt{X(t)} dB(t)
  $$

Used in **interest rate** and **stochastic volatility** modeling.

---

### 7. **Stochastic Volatility Models (e.g., Heston Model)**

* The volatility itself follows a continuous-time process:

  $$
  dV_t = \kappa(\theta - V_t)dt + \sigma \sqrt{V_t} dW_t
  $$

  with asset price:

  $$
  dS_t = \mu S_t dt + \sqrt{V_t} S_t dB_t
  $$

---

Each of these has different use cases depending on whether you want **memory**, **boundedness**, **jumps**, or **non-Gaussian noise**.
