We appreciate the constructive feedback and the opportunity to clarify the contributions and motivations behind our work. Below, we address the reviewer’s comments in detail.

## **1. Writing Improvements and Clarification of Table 1**

Thank you for highlighting areas where the writing can be improved. We recognize that Table 1 and its associated paragraphs might not have been as clear as intended. The table demonstrates the comparative performance of different speculative decoding methods, including our Sorted Speculative Decoding (S2D) and self-speculative decoding approaches. Specifically, it highlights that self-speculative decoding methods—where intermediate sub-models of the target are used as drafts—suffer from significant accuracy degradation (15% lower on GSM8K) and slower decoding speed due to the latency of intermediate sub-models. This underscores the motivation for developing a separate draft model, which maintains high accuracy while achieving better speedups.

To address this concern, we will revise the text to make these implications clearer and explicitly explain the significance of the results in Table 1.

Regarding the abstract and introduction, we agree that emphasizing the “one draft model for different target models of varying sizes” is crucial. We will revise these sections to ensure that our multi-target focus is explicitly stated and its practical implications are highlighted.

**2. Practical Need for Multi-Target Handling**

We appreciate the reviewer’s skepticism about the necessity of handling multiple target models. While it is true that many real-world deployments focus on a single large language model (LLM), we believe the multi-target scenario is becoming increasingly relevant due to the following considerations:

***Dynamic Deployment Needs:***
Real-world environments often demand adaptability due to resource constraints or latency requirements. For instance:

In edge AI settings, devices with limited compute resources might require smaller models (e.g., Vicuna-7B) for faster response times, while cloud servers might employ larger models (e.g., LLaMA-70B) for higher accuracy.
Organizations managing diverse deployment environments may prefer a unified draft model that can serve multiple target models, reducing operational complexity.

***Emerging Trends in Many-in-One Models:***
Recent research (e.g., FLEXTRON: Many-in-One Flexible Large Language Model [1]) emphasizes the need for multi-target handling to optimize performance across diverse deployment scenarios. These trends suggest that multi-target setups, which require draft models capable of supporting various target models, are increasingly practical and forward-looking.

***Efficiency of a Single Draft Model:***
As shown in Table 6 (Appendix), training a separate draft model for each target incurs substantial training and memory costs, especially for larger target models. In contrast, our SoFT draft model is trained once and reused across all targets, offering significant efficiency gains without compromising accuracy or speed.

**3. Motivation for Multi-Scale Targets**
The reviewer raises a valid point about the perceived rarity of switching between significantly different target model sizes (e.g., LLaMA-70B to LLaMA-13B). However, we argue that such scenarios are not uncommon in the following contexts:

***Cost-Conscious Deployments:***
Organizations often switch between smaller and larger models based on task complexity and budget constraints. For example, smaller models might handle routine queries, while larger models address more nuanced tasks requiring higher accuracy.

***Flexibility in Deployment Pipelines:***
Developers and researchers often experiment with a variety of model sizes to balance accuracy, latency, and compute requirements. Our single draft model simplifies this workflow by providing a consistent speculative decoding mechanism across all target sizes.
While specific applications may favor model switching within the same size class (e.g., LLaMA-13B to CodeLLaMA-13B), our method is compatible with such setups as well. Moreover, it addresses the broader need for flexibility across both similar and varied model sizes.

**4. Addressing the Preference for Target-Specific Drafts**
The reviewer suggests that training a specific draft model for each target might yield better accuracy and inference speed. While this is theoretically true, it comes at significant operational and computational costs:

***Training Overhead:***
Training a separate draft model for each target requires maintaining and optimizing multiple models, which is not scalable for organizations managing a large fleet of LLMs. As shown in Table 6, methods like Medusa incur much higher memory consumption and fail to scale to larger target models (e.g., LLaMA-70B).

***Deployment Complexity:***
Deploying multiple draft models increases storage, maintenance, and integration complexity. A single, reusable draft model like SoFT simplifies deployment pipelines while delivering competitive accuracy and speed.
Our method balances accuracy, speed, and scalability, providing a practical solution for real-world scenarios where deployment simplicity and efficiency are prioritized.

**5. Novelty and Contribution**

We appreciate the reviewer’s assessment of our work as reasonable and solid. While we acknowledge that our contributions may not be groundbreaking, we believe they are significant in addressing practical challenges in speculative decoding:

***Unified Multi-Target Drafting:***
By introducing a single draft model for multiple target models, we address scalability and deployment challenges in multi-model environments.

***Sorted Speculative Decoding (S2D):***
Our adaptive draft strategy combines sorted fine-tuning with confidence-based mechanisms, improving the trade-off between decoding speed and accuracy compared to existing methods.

***Comprehensive Evaluation:***
We conducted extensive experiments on MT-Bench and GSM8K, showcasing the efficacy of our method across diverse scenarios. Our ablation studies further validate the benefits of sorted speculative decoding.
We believe these contributions offer meaningful advancements in speculative decoding and its application to multi-target scenarios.

**6. Planned Revisions**

Based on your feedback, we will:

- Clarify the implications of Table 1 and its associated text.
- Emphasize the multi-target capability of our method in the abstract and introduction.
- Highlight practical scenarios and emerging trends that motivate the need for multi-target handling.
- Acknowledge the trade-offs between single and multi-target drafts while reinforcing the scalability benefits of our approach.

[1] Cai et al., "Flextron: Many-in-One Flexible Large Language Model" (ICML 2024)

***We kindly ask the reviewer to reconsider their score based on the clarifications and additional evidence provided, which highlight the scalability, practicality, and robust performance of our method.***
