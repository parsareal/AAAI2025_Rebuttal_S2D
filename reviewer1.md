Thank you for your thoughtful feedback. We appreciate the opportunity to clarify our motivations and approach in designing the multi-target scenario and our choice to prioritize sorted speculative decoding (S2D) over self-speculative methods. Below, we address each point raised:

## **1. Multi-Target Scenario Justification:**

We acknowledge the reviewer’s concerns regarding the necessity of a single draft model serving multiple target models. However, this approach aligns with recent trends in many-in-one frameworks, such as FLEXTRON [ICML 2024], which aim to optimize efficiency and adaptability across diverse deployment scenarios. The multi-target setup is particularly valuable in dynamic environments where accuracy, latency, and resource constraints can vary.

***Practical Utility:*** A single draft model reduces deployment complexity by enabling seamless transitions between target models of varying sizes and requirements. This adaptability eliminates the need for multiple target-specific drafts, saving time, memory, and operational costs.

***Emerging Trends:*** Multi-target setups are increasingly relevant in edge AI and cloud environments, where different models might be deployed based on task complexity or hardware constraints.

## **2. Advantages of Sorted Speculative Decoding (S2D) Over Self-Speculative Decoding:**

We have compared S2D with self-speculative decoding using nested intermediate sub-models of the target model (Table 1) and early-exiting mechanisms (Table 2). These comparisons highlight key limitations of self-speculative methods:

***Accuracy Degradation:*** Nested sub-models lead to a 15% accuracy drop on GSM8K compared to S2D. This is due to perturbations in the target model’s intermediate layers, which compromise its overall performance.

***Lower Speedups:*** Self-speculative methods experience higher latency when speculating tokens with intermediate sub-models, resulting in lower speedups than S2D.

|                                | **GSM8K**                          |                      |
|--------------------------------|------------------------------------|----------------------|
| **Model**                      | **Auto-regressive Decoding**       |                      |
|                                | **Speedup**                        | **Accuracy**         |
| SFT (Llama2 13B)               | 1×                                 | 48.97                |
|                                | **Self Sorted Speculative Decoding (Sorted Target)** | |
| **Model**                      | **Speedup**                        | **Accuracy**         |
| Layers 12:40 (SoFT)            | 1.21×                              | 33.51                |
|                                | **Sorted Speculative Decoding (Sorted Draft)** |   |
| **Draft Model**                | **Speedup**                        | **Accuracy**         |
| Layer 6:12 (SoFT 6,9,12 13B)   | **1.53×**                          | 48.97                |

**Table:** Performance comparison between self sorted speculative decoding (sorted target) and sorted speculative decoding (sorted draft) proposed in this paper on GSM8K dataset.

In Table 2, we assess early-exiting mechanisms as an alternative self-speculative approach. While they avoid altering the target model’s weights, they yield significantly lower Mean Accepted Tokens (MAT) and speedups compared to S2D. This highlights the superiority of a dedicated draft model for speculative decoding.

| **Method**                  | **Vicuna 7B (T=0)** |        | **Vicuna 13B (T=0)** |        | **LLaMA Chat 70B (T=0)** |        | **Vicuna 7B (T=1)** |        | **Vicuna 13B (T=1)** |        | **LLaMA Chat 70B (T=1)** |        | **Avg Speedup** |
|-----------------------------|---------------------|--------|----------------------|--------|--------------------------|--------|---------------------|--------|----------------------|--------|--------------------------|--------|-----------------|
|                             | **Speedup**         | **MAT** | **Speedup**          | **MAT** | **Speedup**              | **MAT** | **Speedup**         | **MAT** | **Speedup**          | **MAT** | **Speedup**              | **MAT** |                 |
| SFT + EarlyExit L6 + SD     | 0.98×              | 1.51   | 1.00×               | 1.51   | 1.32×                  | 1.46   | 0.96×              | 1.52   | 1.03×               | 1.51   | 1.37×                  | 1.53   | 1.11            |
| SFT + EarlyExit L9 + SD     | 1.08×              | 1.99   | 1.05×               | 1.98   | 1.54×                  | 1.80   | 0.99×              | 2.04   | 1.12×               | 2.04   | 1.57×                  | 1.86   | 1.22            |
| **SoFT + S2D (ours)**       | **1.34×**              | 2.86   | **1.38×**           | 2.76   | **1.95×**              | 2.36   | 1.27×              | 3.01   | **1.38×**           | 2.89   | **1.98×**              | 2.44   | **1.55**        |

**Table:** Speedup and MAT results comparing early-exiting and S2D in multi-target setups.

## **3. Training Cost Comparison:**

Medusa, like other target-dependent models such as Eagle and Hydra, requires loading the full target model into memory during training to align the draft with the target outputs. This dependency results in significantly higher memory consumption that scales with target model size, making these methods impractical for larger models like LLaMA2 Chat 70B. As shown below, Medusa encounters out-of-memory (OOM) errors with such large targets.

In contrast, our SoFT Draft is target-independent and requires a single training phase, independent of specific target models. This fixed training process ensures a significantly lower and consistent memory footprint across all target sizes.

| **Method/Target Model**    | **Vicuna 7b** | **Vicuna 13b** | **LLaMA2 Chat 70b** |
|----------------------------|---------------|----------------|---------------------|
| SoFT Draft (12 Layers)     | 92,744 MB     | 92,744 MB      | 92,744 MB           |
| Medusa                     | 113,416 MB    | 152,616 MB     | OOM                 |

**Table:**  Comparison of memory footprints of SoFT draft (12 layers) versus Medusa across different target model sizes. Note: SoFT Draft requires a single training, whereas Medusa must be retrained for each target model.

By removing the dependency on target-specific alignments, SoFT draft not only achieves better scalability but also reduces operational complexity and resource requirements. This makes it an ideal solution for scenarios where multiple target models need to be served efficiently.

## 4. **Clarification on Table 5 (Ablation Study):**

The ablation study in Table 5 aims to demonstrate that training both the target and draft models on a domain-specific dataset can enhance speedup. This result, however, does not pertain to our multi-target setup, where we design a single draft model to accommodate multiple targets. In target-dependent methods like Eagle and Medusa, even if domain-specific training is unnecessary, the draft model must still be aligned with the target’s outputs, imposing constraints that SoFT overcomes by serving multiple targets independently.

| **Model**                    | **GSM8K**                               |                      |                      |
|------------------------------|-----------------------------------------|----------------------|----------------------|
|                              | **Auto-regressive Decoding**            |                      |                      |
|                              | **Trained Target**                      | **Speedup**          | **Accuracy**         |
| Llama2 13B                   | ✔️                                     | 1×                   | 48.97                |
| Llama2 13B                   | ✖️                                     | 1×                   | 28.7                 |
|                              | **Sorted Speculative Decoding**         |                      |                      |
| **Draft Model**              | **Trained Target**                      | **Speedup**          | **Accuracy**         |
| S2D - SoFT                   | ✔️                                     | **1.53×**            | 48.97                |
| S2D - SoFT                   | ✖️                                     | 1.38×                | 28.7                 |

**Table:** Comparison of speedup between two different settings: 1) training the target model on the downstream task 2) using the vanilla pre-trained model

***We respectfully ask the reviewer to consider this clarification and re-evaluate the paper’s contributions.***
