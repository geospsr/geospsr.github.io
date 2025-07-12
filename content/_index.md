+++
title = "Stochastic Poisson Surface Reconstruction with One Solve using Geometric Gaussian Processes"
[extra]
authors = [
    {name = "Sidhanth Holalkere", url = "https://sholalkere.github.io"},
    {name = "David Bindel", url = "https://www.cs.cornell.edu/~bindel/"},
    {name = "Silvia Sellán", url = "https://www.silviasellan.com/"},
    {name = "Alexander Terenin", url = "https://avt.im/"},
]
venue = {name = "ICML 2025", url="https://icml.cc/"}
buttons = [
    {name = "Arxiv", url = "https://arxiv.org/abs/2503.19136"},
    {name = "PDF", url = "https://arxiv.org/pdf/2503.19136"},
    {name = "Code", url = "https://github.com/sholalkere/GeoSPSR"},
]
katex = true
large_card = false
favicon = false
+++

{{ figure(src=["scorp.png"], alt=["Statistical queries such as in probability computed by our method."], body="Our method improves the stochastic reconstruction technique of [Sellán and Jacobson](https://dl.acm.org/doi/10.1145/3550454.3555441).") }}

Surface reconstruction algorithms are used to convert point cloud data—the most common format in which real-world scans are captured—into other downstream-usable formats, such as meshes or other shape representations. For an orientable surface `$\Omega$`, they take in a set of oriented points `$(x_i, v_i)_{i=1}^N$` sampled from the surface and attempt to produce a shape representation of `$\Omega$`.

### Uncertainty Quantification for Surface Reconstruction

#### Poisson Surface Reconstruction
Poisson surface reconstruction (PSR) is the most widely-used algorithm for this task. It returns a surface as an implicit function `$f$`, i.e. `$f$` is negative on the inside of the surface, `$0$` on the surface, and positive on the outside of the surface. It does so by creating a dense, global mesh encapsulating the input points, and then using finite elements to perform the Poisson solve of `$\Delta f(x) = \nabla \cdot v(x)$` on this mesh. 

#### Stochastic Poisson Surface Reconstruction
However, when observations are sparse, surface reconstruction becomes an undetermined problem so instead of reasoning about *one* surface, we may want to consider the *distribution* of surface the input might have been sampled from. As PSR is unable to answer this, [Sellán and Jacobson](https://dl.acm.org/doi/10.1145/3550454.3555441) introduce a statistical extension, Stochastic PSR, which adds these capabilities. Instead of a deterministic `$v$`, they give it a Gaussian process prior `$v \sim \text{GP}(0, k)$` and compute the conditional distribution of `$f \mid v$` which is also Gaussian. The posterior is Gaussian because the Poisson solve is linear and Gaussian processes are closed under linear transformations.

While SPSR has new statistical capabilities, it inherits and introduces a few limitations:

1. it requires global evaluation and is therefore not output-sensitive,
1. it introduces approximations which make covariance calculations inaccurate for realistic length scales, and
1. it couples the discretization structure and kernel length scale which causes runtime to exponentially scale with reconstruction resolution.

### Avoiding the Poisson Solve using Geometric Gaussian Processes
As Gaussian processes are closed under linear transformations, we can directly describe the posterior with:
```
$$
\mu_{f\mid v}(\cdot) = \bold{K}_{f(\cdot)\textit{\textbf{v}}}(\bold{K}_{\textit{\textbf{vv}}} + \Sigma)^{-1} \textit{\textbf{v}}
$$
```
and 
```
$$
k_{f\mid v}(\cdot, \cdot ') = k_f(\cdot, \cdot ') - \bold{K}_{f(\cdot)\textit{\textbf{v}}}(\bold{K}_{\textit{\textbf{vv}}} + \Sigma)^{-1} \bold{K}_{\textit{\textbf{v}} f(\cdot ')}
$$
```
Note that these expressions involve matrices generated from the cross-covariance `$k_{f,v}$` and the scalar field covariance `$k_f$`, but it is not immediately clear how to derive these from the vector field covariance `$k_v$`.

We use the theory of geometric Gaussian processes to compute these. Our main result is that if `$k_v$` is the product of kernels for each dimension: `$k_v(x, x') = \prod_{i=1}^d k_{v_i}(x, x')$` each with Mercer expansions `$k_{v_i}(x, x ') = \sum_{n \in \mathbb{Z}^d} \rho_{v_i}(n)(\sin(\langle n, x \rangle) \sin(\langle n, x ' \rangle) + \cos( \langle n, x \rangle) \cos(\langle n, x ' \rangle))$` then we have 
```
$$
k_{f, v_i}(x, x ') = \sum_{n \in \mathbb{Z}^d \setminus 0} \frac{n_i \sqrt{\rho_{v_i}(n)}}{\|n\|^2} \sin(n \cdot (x - x'))
$$
```
and 
```
$$
k_f(x, x') = \sum_{n \in \mathbb{Z}^d \setminus 0} \frac{\sum_{i=1^d} n_i^2 \rho_{v_i}(n)}{\|n\|^4} (\sin(\langle n, x \rangle) \sin(\langle n, x ' \rangle) + \cos( \langle n, x \rangle) \cos(\langle n, x ' \rangle)).
$$
```
This allows us to evaluate the posterior without any finite element solves. Note that while these expressions consist of infinite sums, we describe strategies to truncate and amortize their cost in the paper.

Furthermore our approach allows for an analytic description of the Gaussian process induced by the Poisson solve. This enables new capabilities, such as directly sampling from the posterior via pathwise conditioning as we can now write
```
$$
(f \mid \textit{\textbf{v}})(\cdot) = f(\cdot) + \bold{k}_{f(\cdot) \textit{\textbf{v}}} (\bold{K}_{\textit{\textbf{vv}}} + \Sigma)^{-1}(\textit{\textbf{v}} - v(\textit{\textbf{x}}) - \varepsilon).
$$
```

### Demonstrations and Examples
{{ figure(src=["collision.png", "falcon.png"], alt=["Collision probabilities of various cats and a dragon.", "Posterior samples along a ray looking at a falcon."], body="We can perform the same tasks as [Sellán and Jacobson](https://dl.acm.org/doi/10.1145/3550454.3555441), such as collision detection and ray casting.
") }}


{{ figure(src=["sampling.png"], alt=["Samples of the posterior given a partial scan of a bunny."], body="Unlike [Sellán and Jacobson](https://dl.acm.org/doi/10.1145/3550454.3555441), we can sample the posterior at high resolutions
") }}

{{ figure(src=["dragon_lengthscale.png"], alt=["Runtime of our method and prior method for a variety of length scales."], body="Out method is faster and scales independently of the kernel length scale
") }}


### Conclusion
We re-examing the statistical extension of PSR introduced by [Sellán and Jacobson](https://dl.acm.org/doi/10.1145/3550454.3555441) with a more mathematically principled approach using geometric Gaussian processes. The proposed method was shown to support the same set of statistical queries as prior work, and additionally provide new, random-sampling-based queries which take into account smoothness properties captured by the kernel’s correlations. Our work constitutes a first step to
incorporating sample-efficient data acquisition techniques from the Gaussian process and Bayesian optimization literatures into surface reconstruction and computer graphics.
