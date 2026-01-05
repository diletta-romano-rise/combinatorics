## Note (PDF typo)

In the PDF there is an error in the definition of when the transient phase ends: I forgot to add **+ (k-1)**.

So the intended threshold is:

$$
n_{\text{end}}(k)=\frac{k(k+1)}{2} + (k-1).
$$

---

# Efficient Computation of k-Subset-Sum Distributions via Translation Invariance

This repository contains code and experiments about an empirical phenomenon (and a resulting algorithm) for computing **k-subset-sum distributions** efficiently.

---

## TL;DR

Define
$$
f(k,n,s)=\bigl|\{\,A\subseteq\{0,1,\dots,n\} : |A|=k,\ \sum_{a\in A} a = s\,\}\bigr|.
$$

i.e., the number of **k-element subsets (no repetitions)** of $\{0,1,\dots,n\}$ with sum $s$.

Main experimental observation:

- If we apply a **nested finite-difference operator** to the sequence $s \mapsto f(k,n,s)$ using offsets $1,2,\dots,k$
  (i.e., apply $\Delta_1$, then $\Delta_2$, …, up to $\Delta_k$),
  the resulting profile $g_{k,n}(s)$ becomes **translation-invariant** as $n$ varies (for fixed $k$).

Algorithmic consequence:

- To jump from $n$ to a larger $m>n$, we can often:
  1. **shift** the stabilized difference profile by a deterministic amount,
  2. **invert** the differences (“integrate back”) to reconstruct $f(k,m,\cdot)$,

  yielding a method that is substantially faster than a standard dynamic-programming (DP) baseline in the tested regimes.

---

## Definitions

### k-subset-sum distribution

- Universe: $\{0,1,\dots,n\}$
- Subsets are **without repetition**
- Goal: the count distribution over sums $s$

### Nested differences (increasing offsets)

Given a sequence/vector $x[s]$ (here $x[s]=f(k,n,s)$):

- $\Delta_j x[s] = x[s] - x[s-j]$ (for $s\ge j$; out-of-range indices are treated as $0$)
- Nested operator:

$$
g_{k,n} = (\Delta_k\circ\Delta_{k-1}\circ\cdots\circ\Delta_1)\,f(k,n,\cdot).
$$

---

## Empirical patterns (from the experiments)

### 1) Transient phase

Empirically, the transient regime ends at

$$
n_{\text{end}}(k)=\frac{k(k+1)}{2} + (k-1).
$$

(See the note above: the PDF version missed the $+(k-1)$ term.)

---

### 2) Pattern for shifts

For fixed $k$, the nested-difference profile $g_{k,n}$ is (up to boundary effects) translation-invariant as $n$ varies.

The peak locations follow a deterministic shift rule when moving from $n$ to $n+1$:

- the **1st peak** shifts by **1**,
- the **2nd peak** shifts by **2**,
- the **3rd peak** shifts by **3**,
- and so on.

---

### 3) Pattern for peaks

#### Number of peaks

Empirically, the number of peaks depends only on $k$ and follows:

 $$
P(k)=
\begin{cases}
2\left\lfloor \dfrac{k}{2}\right\rfloor - 1, & \text{if \(k\) is even},\\
2\left\lfloor \dfrac{k}{2}\right\rfloor, & \text{if \(k\) is odd}.
\end{cases}
$$
### Peak shapes

Empirically, the peak shapes match (up to translation) the distributions for subset sizes

$$
1,2,\dots,\left\lfloor \frac{k}{2}\right\rfloor
$$

computed on the ground set of size $k-1$ (i.e., on $\{0,1,\dots,k-1\}$).

Equivalently, the peaks in $g_{k,n}$ coincide with translated copies of:

$$
f(1,k-1,\cdot),\ f(2,k-1,\cdot),\ \dots,\ f(\lfloor k/2\rfloor,\,k-1,\cdot).
$$

---

## Reconstruction

The reconstruction strategy is:

1. Compute the exact distribution $f(k,n,\cdot)$ for a (smaller) value of $n$ (e.g., via a DP baseline).
2. Apply nested differences to obtain

$$
g_{k,n} = (\Delta_k \circ \cdots \circ \Delta_1)\,f(k,n,\cdot).
$$

3. Shift the peak structure to move from $n$ to a larger $m$, using the deterministic rule:
   - the 1st peak shifts by $1$ per $+1$ in $n$,
   - the 2nd peak shifts by $2$ per $+1$ in $n$,
   - etc.

   This produces an estimated (shifted) profile $g_{k,m}$.

4. Reconstruct $f(k,m,\cdot)$ from $g_{k,m}$ by inverting the nested differences (“integrate back”).

---

## How to write it (difference-equation form)

Instead of explicitly “integrating back” the differences, we can describe the reconstruction using an explicit linear difference equation.

Example (explicit recurrence form):

$$
x(i-6) - x(i-5) - x(i-4) + x(i-2) + x(i-1) = 0.
$$

Then we add the contribution of the shifted peaks (i.e., translated peak patterns) to obtain the full reconstruction.
