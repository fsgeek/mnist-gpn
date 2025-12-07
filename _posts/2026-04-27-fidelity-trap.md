---
layout: distill
title: "The Coverage Boundary: Why High-Fidelity Primitives Don't Compose"
description: "A controlled experiment showing that adversarially trained primitives hit a glass ceiling on compositional generalization, while low-fidelity pedagogical primitives achieve perfect transfer."
date: 2026-04-27
future: true
htmlwidgets: true
authors:
  - name: Anonymous
tags: [compositionality, generalization, neural networks, representation learning]

bibliography: 2026-04-27-fidelity-trap.bib

toc:
  - name: The Question
  - name: "Background: Coverage and Compositionality"
  - name: The Experiment
  - name: Experimental Architecture & Controls
  - name: The Fidelity Trap
  - name: The Coverage Boundary
  - name: "The Showdown: The Glass Ceiling of Adversarial Training"
  - name: "Why It Matters: Contract of Appearance vs. Contract of Meaning"
  - name: Summary
  - name: Related Work
  - name: Limitations & Next Steps
---

## The Question

Can neural networks learn compositional rules that generalize beyond their training data?

The compositional generalization literature has established that models can succeed on _novel combinations of known primitives_: a system that learns "red circle" and "blue square" can often generate "red square" <d-cite key="keysers2020cfq,park2021benchmark"></d-cite>. But this hides a critical assumption:

> **What exactly counts as a "known" primitive?**

Consider a generative model pre-trained to produce images of the digit 7. Does high visual fidelity, the ability to render a photorealistic 7, constitute "knowing" the digit well enough to use it compositionally in relational tasks like "generate X > Y”?

Intuitively, we assume better primitives yield better composition. If a model can generate a crisp, perfect digit, it must understand that digit.

**We found the opposite.**

In a controlled experiment, we show that high-fidelity primitives trained adversarially (GANs) hit a _glass ceiling_ of composability, while low-fidelity "blotchy" primitives trained pedagogically achieve perfect transfer.

---

## Background: Coverage and Compositionality

Recent work has clarified that compositional generalization is constrained by the _coverage_ of primitives in training data. Benchmarks like SCAN and its descendants show that models struggle on held-out combinations when key primitives never appear in the right structural contexts <d-cite key="keysers2020cfq"></d-cite>. The **Coverage Principle** formalizes this: for pattern-matching learners, reliable generalization is only possible within the "coverage" of functionally equivalent fragments seen during training <d-cite key="chang2025characterizingpatternmatchinglimits"></d-cite>. In other words, coverage is a **necessary condition** for compositional generalization.

Our experiments take this as a starting point. We instantiate the Coverage Principle in an intentionally simple generative setting and then ask a deeper question: **even when coverage is satisfied, do all primitives admit compositional use?**

---

## The Experiment

To investigate this boundary, we designed a deliberately simple experiment using **Relational MNIST**. The task: generate three-digit displays of the form `[X][>][Y]` where `X` and `Y` are MNIST-style digits and `X > Y` numerically.

The simplicity is intentional. MNIST is the petri dish, not the ecology. If the coverage boundary failed to appear here, in the most controlled possible environment, it would suggest the phenomenon is an artifact of complexity. That it appears so sharply in this minimal setting implies a fundamental property of neural compositionality that scale may _mask_ but cannot _cure_.

Our approach follows **pedagogical training with frozen primitives**. We pre-trained a _single-digit weaver_ to generate individual digits `[0-9]`, then froze it and trained only a compositional layer, the _latent splitter_, to route latent codes for generating relational displays.

Crucially, we compared two types of teachers for the primitive generator:

1. **Adversarial (GAN):** Optimized to fool a discriminator, producing sharp, high-fidelity digits rich in texture.
2. **Pedagogical (Ours):** Optimized for structural reconstruction, producing abstract, low-frequency representations that preserve structure but discard texture.

{% include figure.liquid path="assets/img/2026-04-27-fidelity-trap/fig4_fidelity_comparison.png" class="img-fluid" %}

<div class="caption">
    Figure 1: The Fidelity Trap. Left: Standard GAN samples, high contrast and sharp edges, but carrying pseudo-texture learned to satisfy an adversarial discriminator. Right: Our pedagogical samples, visually blotchy and diffuse, prioritizing structural clarity over textural noise.
</div>

---

## Experimental Architecture & Controls

We separate _primitive competence_ from _relational competence_ by freezing the primitive generator and training only the relational layer.

- **Single-Digit Weaver (Frozen).** Pre-trained on digits `[0-9]` using either adversarial (GAN) or pedagogical objectives. Once trained, its weights are frozen.
- **Latent Splitter (Trainable).** Receives a latent code and learns to route it into `(X, >, Y)` displays, implementing relational structure over the same primitives.
- **Static Judge (Ground Truth Oracle).** Evaluates whether `X > Y` holds numerically. The judge is fixed and never trained. The judge sees only the rendered digits and not the internal latent codes, preventing any trivial leakage.

Key controls:

- The primitive generator **never** sees the relational test set.
- The relational layer (latent splitter) is trained only on training relations; held-out relations are used purely for evaluation.
- After early experiments revealed collusion when the student could influence the teacher, we removed all student-to-teacher reward paths: the teacher’s objective depends solely on student performance as evaluated by the static judge.
- During all relational experiments, primitive generators are **frozen**, ensuring that differences in performance arise from the training objectives used to build primitives, not from additional fine-tuning.

{% include figure.liquid path="assets/img/2026-04-27-fidelity-trap/fig3_experimental_design.png" class="img-fluid" %}

<div class="caption">
    Figure 2: Experimental architecture. We freeze the primitive generator (whether GAN or Pedagogical) and train only the relational routing layer.
</div>

---

## The Fidelity Trap

A surprising observation emerged during primitive training. The pedagogical teacher generated digits that were visually _blotchy_, diffuse, soft-edged, and structurally abstract.

This apparent degradation acts as a **semantic bottleneck**. By discarding texture, the pedagogical objective forces the latent space to represent only the structural information required for compositional reasoning. Importantly, this does not imply that high-fidelity rendering is undesirable, only that it should be _decoupled_ from structural learning. A plausible training recipe is two-stage: first learn the concept under a “Contract of Meaning” (low fidelity, high structure), then layer the “Contract of Appearance” (high fidelity) only after the compositional logic is secured.

In the discriminative setting, Geirhos et al. famously showed that ImageNet-trained CNNs are strongly biased toward texture, and that increasing shape bias improves robustness and generalization <d-cite key="geirhos2018imagenet"></d-cite>. Our results suggest an analogous phenomenon on the generative side: adversarial objectives encourage texture-rich primitives that look good but compose poorly, whereas pedagogical objectives yield "blotchy" but primitives that compose perfectly.

This matters methodologically: because our primitives are consistent with topology rather than texture, logical failures in the relational task cannot be attributed to pixel-level distribution shift. The model knew the abstract form of "7" perfectly. The only remaining question was whether it could _use_ that knowledge compositionally.

---

## The Coverage Boundary

We first asked whether primitives could compose _without_ specific relational training coverage.

- **Condition (Phase 1.5):**
  Train relational displays only for digits `[0-4]` (10 valid `X > Y` pairs).
  Test on relational displays for digits `[5-9]`, which are **completely unseen** in relational context.

- **Result:**
  **0% digit accuracy** and ~**chance-level relation accuracy** on the novel digits.

The model produced recognizable digits in isolation but garbage in relational contexts. This concretely instantiates the **Coverage Principle** <d-cite key="chang2025characterizingpatternmatchinglimits"></d-cite> in a generative setting:

> **Primitive Competence** (being able to draw a 7)
> **does not grant**
> **Compositional License** (using 7 correctly in a relation).

As the Coverage Principle predicts, license is only acquired when a primitive appears in a _relational_ context during training. Coverage is necessary, but, as we show next, it is not sufficient.

{% include figure.liquid path="assets/img/2026-04-27-fidelity-trap/fig1_coverage_boundary.png" class="img-fluid" %}

<div class="caption">
    Figure 3: The coverage boundary and glass ceiling. Same architecture, different training objectives, opposite outcomes.
</div>

---

## The Showdown: The Glass Ceiling of Adversarial Training

Once we established that coverage is necessary, we asked the deeper question:

> **Is coverage sufficient?**

If we give an adversarial model every advantage, full relational coverage, identical architecture, and visually superior primitives, can it match pedagogical performance?

We ran the experiment on **Novel Combinations** (Phase 1.6):

- **Training:**
  Digits `[0-9]`, with 41 of 45 valid `X > Y` pairs. We hold out four specific relations (e.g., `7 > 3`, `8 > 2`, `9 > 1`, `6 > 4`). Every digit appears in many relational contexts during training.

- **Testing:**
  Only the 4 held-out relations. These are _novel combinations of seen digits_.

**Results:**

| Training Objective | Primitives | Held-out Relation Accuracy |
| ------------------ | ---------- | -------------------------- |
| Pedagogical (Ours) | Blotchy    | **100.0%**                 |
| Adversarial (GAN)  | Crisp      | **81.1%**                  |

Similar symptoms have been reported at scale in text-to-image systems: models can render individual concepts with high fidelity yet catastrophically fail on compositional prompts (negation, counting, spatial relations), even when evaluation metrics like FID remain strong <d-cite key="park2021benchmark,huang2023t2i,vatsa2025rightlookswrongreasons"></d-cite>. These works document the **what**. Our result isolates a candidate **why**: adversarial objectives encourage texture-heavy representations that cannot be perfectly recomposed, even under full relational coverage.

The adversarial model is not "broken." 81% is not failure, it is a _ceiling_. The model had full relational coverage. It had seen every digit in compositional context. Yet it could not fully compose. A natural question is whether this 81% ceiling would disappear at larger scales. While increasing parameters or data might push performance upward by brute-force memorization of more relational pairs, the **structural tax remains**. The pedagogical model reaches 100% with minimal data because its primitive representations are composition-friendly from the start. In contrast, adversarially trained primitives must continually burn capacity to maintain textural fidelity. Thus the “Glass Ceiling” should be interpreted not as an absolute limit at infinite scale, but as a measure of **compositional inefficiency** introduced by adversarial objectives.

This is the **Glass Ceiling of Adversarial Training**. The model pays a **tax on composition**: capacity spent maintaining textural fidelity entangles the latent space in ways that resist perfect reassembly. No amount of additional coverage can break through, because the limitation appears structural rather than statistical.

By contrast, our pedagogical primitives, although visually worse, compose perfectly—consistent with cleaner underlying structure.

---

## Why It Matters: Contract of Appearance vs. Contract of Meaning

This experiment is a critique of how we train generative models.

Modern practice follows a Contract of Appearance. Adversarial objectives (GANs) and preference optimization (RLHF/RLAIF) reward models for producing outputs that match surface statistics—textures, sharpness, or human-rated plausibility. Appearance is not inherently problematic; indeed, it is crucial in many applications. The difficulty emerges when appearance is optimized too early, before the underlying structure is stabilized. Premature optimization entangles texture with structure, forcing a model to satisfy a discriminator’s aesthetic constraints at the same time it is trying to learn a rule. This entanglement imposes a structural tax on composition. As our GAN results show, this produces high-fidelity primitives that look perfect to a critic but are hollow to a composer. They possess **Primitive Competence** but lack **Compositional License**. This same pattern appears in large language models trained with reinforcement learning from human feedback (RLHF/RLAIF): optimizing for human-rated plausibility can privilege surface agreement over structural understanding, with downstream costs to robustness and compositional generalization <d-cite key="vatsa2025rightlookswrongreasons"></d-cite>.

Our pedagogical approach enforces a **Contract of Meaning**. By restricting the model to "blotchy," low-frequency primitives, we create a **semantic bottleneck** that forces the latent space to prioritze invariant structure over texture. The model must learn the concept, the topology of the digit, because the texture is unavailable. High-fidelity appearance could be layered on _after_ struture is learned, but confounding the two objectives during early training degrades compositional generalization.

This distinction matters for safety and robustness. A model that understands meaning can be trusted to handle unseen combinations; a model trained under a Contract of Appearance may look correct while behaving unpredictably outside its training manifold.

Although demonstrated here on MNIST for clarity, the Coverage Boundary and Glass Ceiling are **architectural** phenomena, not dataset quirks. Large-scale generative training (GANs, diffusion, preference tuning) may be subject to the same fidelity trap: objectives that reward appearance can actively degrade compositional reasoning, even when coverage is abundant.

---

## Summary

| Experiment    | What's Novel                              | Result                                      |
| ------------- | ----------------------------------------- | ------------------------------------------- |
| **Phase 1.5** | Novel Primitives (No Relational Coverage) | **0% Transfer** — The Coverage Boundary     |
| **Phase 1.6** | Adversarial Primitives (Full Coverage)    | **81.1% Accuracy** — The Glass Ceiling      |
| **Phase 1.6** | Pedagogical Primitives (Full Coverage)    | **100% Accuracy** — The Contract of Meaning |

The Coverage Boundary tells us _when_ composition is possible.
The Glass Ceiling tells us _whether_ the primitives are capable of it.

You need both: primitives shaped for meaning, and coverage that licenses their use.

Code and experimental details will be released upon acceptance.

---

## Related Work

Our setup connects to several strands of prior work. Compositional generalization benchmarks such as SCAN and its extensions highlight the importance of primitive coverage in sequence-to-sequence models <d-cite key="keysers2020cfq,friedman2022findingdatasetshortcutsgrammar"></d-cite>. The **Coverage Principle** of Chang et al. (2025) formalizes coverage as a necessary condition for pattern-matching learners, a condition our "Coverage Boundary" experiment instantiates in a generative regime <d-cite key="chang2025characterizingpatternmatchinglimits"></d-cite>.

On the generative side, compositional text-to-image benchmarks repeatedly find that models with excellent perceptual quality metrics still fail on novel combinations of attributes and objects <d-cite key="park2021benchmark,huang2023t2i"></d-cite>. Vatsa et al. (2025) describe this as "right looks, wrong reasons," emphasizing failures of compositional fidelity in modern diffusion models <d-cite key="vatsa2025rightlookswrongreasons"></d-cite>. Our **Glass Ceiling** result pinpoints adversarial objectives as one mechanism that can produce this pattern, even in a minimal MNIST petri dish.

Our "blotchy but coherent" primitives resonate with work on shape vs. texture bias in CNNs <d-cite key="geirhos2018imagenet"></d-cite> and with emerging views of deep representations as learning topological manifolds amenable to symbolic or relational reuse. We view our pedagogical objective as a small, controlled example of **training for meaning rather than appearance**, a design choice that may scale to more realistic architectures and datasets.

We view generative compositionality as an underexplored junction between representation learning and training objectives, and hope this minimal example encourages further mechanistic work.

---

## Limitations & Next Steps

**Toy domain.** Our experiments use MNIST to make the phenomenon as visible and controllable as possible. Real-world data are higher dimensional and noisier, but if the fidelity trap appears in this simplest setting, we expect it to persist, if hidden, at scale.

**Topology.** While we infer topology from our results, claiming this will require further validation.

**Frozen primitives.** We freeze the digit generator when training relations to cleanly separate primitive learning from relational learning. Future work could study joint training and analyze how much compositional capacity can be recovered, or destroyed, when primitives continue to adapt.

**Single relation.** We focus on a single relational operator (`>`). Extending to multiple relations (equality, ordering, arithmetic expressions) and to symbolic domains would test whether pedagogical primitives systematically support richer compositional logics.

**Beyond MNIST.** The natural next step is to apply pedagogical objectives to more complex visual and language domains, and to compare them directly against adversarial or preference-based objectives used in modern AI training pipelines.

If the fidelity trap generalizes, then **training models to teach rather than to mimic** may be a necessary ingredient in building systems that truly understand, and safely extend, what they learn.
