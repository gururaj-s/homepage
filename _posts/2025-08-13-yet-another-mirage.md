---
layout: post
author: gururaj
---

## Yet Another Mirage of Breaking MIRAGE  
*Debunking Occupancy-based Side-Channel Attacks on Fully Associative Randomized Caches*  

**Authors:** Chris Cao, Gururaj Saileshwar (University of Toronto)  
\[[Paper](https://gururaj-s.github.io/assets/pdf/Yet-Another-Mirage.pdf)\] \[[Code](https://github.com/sith-lab/yet-another-mirage-of-breaking-mirage)\]

---

### 📌 **TL;DR**
A recent USENIX Security 2025 paper, ["Systematic Evaluation of Randomized Cache Designs against Cache Occupancy" (RCO)](https://www.usenix.org/conference/usenixsecurity25/presentation/chakraborty) claimed that **[MIRAGE](https://www.usenix.org/conference/usenixsecurity21/presentation/saileshwar)** (USENIX Security 2021), a fully associative randomized cache, is vulnerable to cache-occupancy attacks that can leak AES keys. This claim has been reiterated by a subsequent [SoK paper](https://www.usenix.org/conference/usenixsecurity25/presentation/bhatla) at USENIX Security 2025.

We show that these claims are **incorrect**. The reported leakage comes from **modeling bugs** in RCO, and not actual vulnerabilities. When modeled correctly, MIRAGE **does not leak AES keys** via occupancy attacks.  We provide more details below, and in an accompanying [paper]({{ site.baseurl }}/assets/pdf/Yet-Another-Mirage.pdf).

---

### Update (31 Aug 2025):
We discover an additional bug in the RCO's artifact in their GE analysis code (see Bug-1 below). Once we fix that, the results from RCO's raw data now match with our naive reproduction. While the naive reproduction of RCO demonstrates low GE (key leakage), after addressing the modeling bugs in RCO (see Bug-2 below), the GE for MIRAGE remains high even with thousands of AES traces (no key leakage). 

---

#### **Background**

**MIRAGE** ([USENIX Security '21](https://www.usenix.org/conference/usenixsecurity21/presentation/saileshwar)) is a cache design that:
- Emulates a *fully associative* cache with random replacement.
- Evicts cache lines **globally at random** to eliminate set-conflict channels.
- Guarantees set-associative evictions are practically impossible over a system’s lifetime.

**Cache occupancy attacks** differ from set-conflict based attacks like Prime+Probe:
- They measure *aggregate* cache usage (overall occupancy).
- MIRAGE does **not** claim to eliminate such channels, but exploiting them for leaking fine-grained secrets like AES keys is challenging.

The RCO paper claims MIRAGE was especially vulnerable due to its fully associative random eviction policy. We seek to evaluate these claims.

---

#### 🔍 **Finding 1: RCO’s Results Are Not Fully Reproducible**
Using RCO’s publicly released artifact, we reproduced the reported trends for AES key leakage. However, RCO reports AES key guessing entropy (GE) for MIRAGE -- higher is better -- dropping below 80 after just 300 AES traces (Mirage-RCO). In
contrast, our reproduction of their data (MIRAGE-Reproduced) shows the same GE reduction occurs after 1400 traces. 

***Bug-1***: We discovered that this mismatch is due to a bug in RCO's artifact for GE calculation, in how they index traces saved across multiple files, due to which their code uses roughly 6× more traces than intended to calculate GE.

**Figure 1 – Guessing Entropy Reproduction:**  
![Figure 1: Guessing Entropy Reproduction]({{ site.baseurl }}/assets/blog/yet-another-mirage/fig1_guessing_entropy.png)  
> In our reproduction of RCO's artifact, MIRAGE's guessing entropy drops only after 1400 AES encryptions, unlike RCO paper’s reported drop after 300 traces. Once we fix the analysis bug in RCO's artifact, their results match our reproduction. 

---

#### 🔍 **Finding 2: RCO's Claims of Leakage Don’t Hold**

RCO attributes the leakage of the AES-Keys in the T-Table implementation in MIRAGE to:
1. **AES last-round T-Table accesses** impacting cache occupancy of MIRAGE.
2. This cache occupancy is **measurable by a subsequent attacker accessing its own cached array**, originally occupying 50% of the Last-level Cache, and **measuring its access time**.

Thus, RCO claims that an attacker measuring its own access time, can perceive cache occupancy, and uncover the secret key based on its correlation with the cache occupancy.

> However, we find that MIRAGE's random evictions cause significant timing variations across repeated encryptions of the same plaintext and key, making keys or ciphertexts indistinguishable based on timings (see Figure-3 below).

---

#### 🛑 **Finding 3: Modeling Flaw in RCO; Fixed RNG Seed for Evictions**

***Bug-2:*** RCO’s simulation initializes MIRAGE’s global eviction RNG with a **constant seed** for every AES encryption. This produces the **same deterministic sequence of evictions** for every AES encryption.

While Cache Occupancy (*O*) is a function of both Victim Accesses (*V*) and Global Evictions (*GE*), i.e., *O = f(V, GE)* in MIRAGE, RCO's assumption of a deterministic sequence of GEs each time artificially makes *O* trivially correlated with key-dependent memory accesses, *V*. However, this is not realistic as in a real hardware implementation, an attacker cannot reset the RNG state of the global evictions to a fixed state for each AES encryption.

✅ **Our Fix: Correct modeling of MIRAGE requires randomizing the global eviction seed** per encryption (e.g., using `std::random_device` on systems where a hardware-based entropy source is available).

After this fix, no correlations between the victim T-Table accesses in the last round and the attacker access times are observed.

**Figure 2 – Heatmap of Attacker Access Times vs T-Table Accesses in Last AES Round:**  
![Figure 2: Heatmap Correlations]({{ site.baseurl }}/assets/blog/yet-another-mirage/fig2_heatmaps.png)  

This figure shows the attacker access times associated with each T-Table access performed in the last round of AES, with the 256 potential T-Table entries visualized in a 16 x 16 matrix. This heatmap is generated for both the profiled key by the attacker, and the victim key.  

> Left: Fixed seed (RCO modeling flaw) shows strong correlation → leakage.  
> Right: Random seed (our fix) removes correlation → no leakage.

The reason for the lack of correlation with random seeds is explained next.

**Figure 3 – Attacker Access Times for Repeated Encryptions of Different Plaintexts**

![Figure 3: Attacker Access Times]({{ site.baseurl }}/assets/blog/yet-another-mirage/fig3_access_times_histogram.png)


We show the attacker access times for four sample victim AES encryptions (repeated 100 times), with different plaintext-ciphertext pairs, selected at random. Each of these has a distinct AES T-Table access pattern.

With Fixed Seed (RCO) on the left, the access time is a signal that allows the encryptions to be distinguished (latency difference of ~1000 cycles).

With Random Seed (our fix) on the right, the access times for each of the encryptions have a considerable spread of almost 100,000 cycles. Thus, any timing variation due to minor occupancy differences is indecipherable, making the different T-Table sequences indistinguishable for an attacker.

---

#### 🏁 Final Results After Fixing Flaws

**Figure 4 – Guessing Entropy After Fixes:**  
![Figure 4: Guessing Entropy After Fixes]({{ site.baseurl }}/assets/blog/yet-another-mirage/fig4_guessing_entropy_fixed.png)

> After applying our fixes (using Random seed for RNG), we observe that the Guessing entropy for the unknown AES key in  MIRAGE remains high (above 90%), demonstrating no key leakage.

On a side note, we observe that the RCO artifact uses a non-standard 512 **byte** L1 cache, in a departure from the paper, which mentions a 512 **kilobyte** L1 cache. Regardless, for a 512-byte or a more standard 64KB L1 cache size, MIRAGE continues to have high Guessing Entropy when modeled correctly.

---

#### Key Takeaways

- **No AES Key Leakage in MIRAGE:** When modeled correctly, MIRAGE is resilient to occupancy-based AES attacks.
- **Simulation Fidelity Matters:** Fixed RNG seeds can introduce artificial determinism, leading to misleading conclusions.
- **Random Seed for Global Eviction Modeling:** Seeding the global eviction RNG realistically captures MIRAGE’s behavior, showing that cache occupancy lacks the fine-grained visibility needed for practical attacks such as AES key extraction.

For more details on these modeling flaws, please see our accompanying [paper]({{ site.baseurl }}/assets/pdf/Yet-Another-Mirage.pdf)

---

#### References

1. [MIRAGE: Mitigating Conflict-Based Cache Attacks with a Practical Fully-Associative Design](https://www.usenix.org/conference/usenixsecurity21/presentation/saileshwar), USENIX Security 2021.  
2. RCO: [Systematic Evaluation of Randomized Cache Designs against Cache Occupancy](https://www.usenix.org/conference/usenixsecurity25/presentation/chakraborty), USENIX Security 2025.  
3. SoK: [So, You Think You Know All About Secure Randomized Caches?](https://www.usenix.org/conference/usenixsecurity25/presentation/bhatla), USENIX Security 2025.

---
