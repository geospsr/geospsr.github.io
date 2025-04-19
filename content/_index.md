+++
title = "Stochastic Poisson Surface Reconstruction with One Solve using Geometric Gaussian Processes"
[extra]
authors = [
    {name = "Sidhanth Holalkere", url = "https://sholalkere.github.io"},
    {name = "David Bindel", url = "https://www.cs.cornell.edu/~bindel/"},
    {name = "Silvia Sellán", url = "https://www.silviasellan.com/"},
    {name = "Alexander Terenin", url = "https://avt.im/"},
]
venue = {name = "Under Review"}
buttons = [
    {name = "Arxiv", url = "https://arxiv.org/abs/2503.19136"},
    {name = "PDF", url = "https://arxiv.org/pdf/2503.19136"},
    {name = "Code", url = "https://github.com/sholalkere/GeoSPSR"},
]
katex = true
large_card = false
favicon = false
+++

{{ figure(src=["teaser.png"], alt=["Comparison of our method and SPSR on a Dragon mesh."], body="Our method computes posteriors faster and more accurately than previous ones, especially at smaller length scales.") }}

# Abstract

Poisson Surface Reconstruction is a widely-used algorithm for reconstructing a surface from an oriented point cloud. To facilitate applications where only partial surface information is available, or scanning is performed sequentially, a recent line of work proposes to incorporate uncertainty into the reconstructed surface via Gaussian process models. The resulting algorithms first perform Gaussian process interpolation, then solve a set of volumetric partial differential equations globally in space, resulting in a computationally expensive two-stage procedure. In this work, we apply recently-developed techniques from geometric Gaussian processes to combine interpolation and surface reconstruction into a single stage, requiring only one linear solve per sample. The resulting reconstructed surface samples can be queried locally in space, without the use of problem-dependent volumetric meshes or grids. These capabilities enable one to (a) perform probabilistic collision detection locally around the region of interest, (b) perform ray casting without evaluating points not on the ray's trajectory, and (c) perform next-view planning on a per-slice basis. They also improve reconstruction quality, by not requiring one to approximate kernel matrix inverses with diagonal matrices as part of intermediate computations. Results show that our approach provides a cleaner, more-principled, and more-flexible stochastic surface reconstruction pipeline.
