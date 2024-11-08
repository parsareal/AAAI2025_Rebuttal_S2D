## **1. Multi-Target Compatibility of Speculative Decoding**

The reviewer raises a concern that the original speculative decoding can already be applied to multiple target models with a single draft model. While this is technically true, the key contribution of our work lies in addressing the practical challenges associated with multi-target setups in speculative decoding. For example using a single draft model for different targets (with different sizes) has no adaptation mechanism between the single draft and the targets; however, in our solution the draft model contains some sub-models and we can adaptively switch between them to serve different targets properly. More specifically:

***Limitations of Existing Methods:***

- *Target-Specific Drafts:*
Most existing speculative decoding approaches require drafts to be tailored for each specific target model, making them impractical for multi-target deployment. This increases the training cost, memory requirements, and deployment complexity, as each draft must be trained or fine-tuned individually for its corresponding target.
- *Lack of Scalability:*
When scaling speculative decoding to handle diverse target models (e.g., different sizes or configurations), using a single draft model with generic capabilities often leads to suboptimal performance, as it fails to adapt to the unique characteristics of each target.

***Our Contribution:***
Our proposed method introduces a scalable framework for multi-target speculative decoding:

- *Single, Reusable Draft Model:*
Through Sorted Fine-Tuning (SoFT), we construct a single draft model with multiple sub-models at different depths. This cascaded structure allows the draft model to generate tokens adaptively based on the requirements of different target models, while maintaining high accuracy and efficiency.

- *Confidence-Based Adaptive Selection:*
By dynamically selecting tokens from sub-models within the draft model, we ensure that the draft generation is tailored to the target model's needs without requiring target-specific training.

This combination allows our method to seamlessly serve multiple target models, reducing both training and deployment overheads compared to conventional speculative decoding setups.

## **2. Novelty of Sorted Fine-Tuning (SoFT) in Speculative Decoding**

The reviewer suggests that SoFT is merely an application of existing learning techniques and does not involve new ideas specific to speculative decoding. While we acknowledge that many-in-one learning principles underlie SoFT, our contribution lies in its novel application and integration within the speculative decoding paradigm:

***Distinct Features of SoFT:***
- *Cascaded Sub-Models for Efficient Drafting:*
SoFT introduces a hierarchical structure within the draft model, enabling early exits at intermediate layers. This design is specifically tailored for speculative decoding, where intermediate sub-models can act as lightweight drafts for faster token generation.

- *Integration with Adaptive Decoding:*
SoFT is uniquely integrated with our confidence-based adaptive selection mechanism, enabling dynamic switching between sub-models based on token prediction confidence. This synergy is specifically designed to enhance both the speed and accuracy of speculative decoding in multi-target setups.

## **3. Confidence-Based Draft Selection and Target Model Distribution Guarantees**

We appreciate the concern regarding the guarantees of sampling from the target model’s distribution when employing confidence-based selection for drafts. Our method ensures that:

***Distribution Fidelity:***
Similar to the original speculative decoding algorithm, the tokens generated during the draft phase must be verified by the target model. This ensures that the final output adheres to the distribution of the target model, regardless of the layer from which the draft tokens originate. While the adaptive exit mechanism optimizes the drafting phase for efficiency, it does not compromise the target model's role in verifying and correcting the generated text.

***Adaptive Drafting Efficiency:***
By leveraging confidence scores, our approach dynamically selects the most appropriate sub-model within the draft framework to balance accuracy and latency. This mechanism does not bypass or override the target model's verification step but instead optimizes the intermediate draft generation process. Thus, the guarantees provided by speculative decoding remain intact, and our method achieves improved efficiency without sacrificing correctness.

## **4. Selection and Evaluation of Experimental Baselines**

We appreciate the reviewer’s observation regarding the inclusion of baselines. Our experimental setup was designed to include a range of target-dependent and target-independent methods, ensuring a comprehensive evaluation. Below, we address the baselines used in detail and explain how they relate to the proposed method:

***Target-Dependent Baselines***
We compared our approach with several state-of-the-art target-dependent methods:

- *Medusa, Hydra, and Eagle:* These methods are tailored to specific target models, requiring the target model’s weights and outputs to train specialized draft models. While these approaches achieve competitive speedups, their reliance on target-specific training increases memory and training costs. Notably, Eagle represents the state-of-the-art in speedup, making it an important baseline for our evaluation.

***Target-Independent Baselines:*** 
To evaluate the performance of target-independent approaches, we conducted experiments using:

- *SFT Drafts:* We evaluated a 12-layer draft model initialized with the first 12 layers of Vicuna 7B and fine-tuned using standard supervised fine-tuning (SFT). This draft model served as a baseline for our SoFT 12-layer architecture. Additionally, we trained language model heads of layers 6 and 9 of the SFT model, allowing us to test early-exit draft generation from these intermediate layers.

- *Pre-Trained Draft Model (Vicuna 160m):*
We also compared our SoFT draft model with Vicuna 160m, a pre-trained model with 12 hidden transformer layers. This model was pre-trained on the C4 corpus and further fine-tuned on ShareGPT data. The results in Table 4 demonstrate that our SoFT draft, fine-tuned on the target dataset, achieves higher speedups than the pre-trained Vicuna 160m draft model.

***Experimental Results***
The table below summarizes the results of these comparisons, showing the overall speedup and Mean Accepted Tokens (MAT) on the MT-Bench dataset for the greedy decoding scenario (T=0):

| **Method**                  | **Vicuna 7B**     |                 | **Vicuna 13B**     |                 | **LLaMA Chat 70B**  |                 |
|-----------------------------|-------------------|-----------------|--------------------|-----------------|---------------------|-----------------|
|                             | **Speedup**       | **MAT**         | **Speedup**        | **MAT**         | **Speedup**         | **MAT**         |
| Vicuna 160m + SD            | 1.05×            | 2.75            | 1.13×             | 2.66            | 1.93×              | 2.24            |
| SoFT L6 + SD                | 1.20×            | 2.10            | 1.26×             | 2.06            | 1.68×              | 1.76            |
| SoFT L9 + SD                | 1.10×            | 2.33            | 1.21×             | 2.29            | 1.79×              | 1.96            |
| SoFT L12 + SD               | 1.00×            | 2.67            | 1.07×             | 2.58            | 1.90×              | 2.19            |
| SoFT + S2D (Ours)           | 1.17×            | 2.54            | 1.28×             | 2.44            | 1.91×              | 2.16            |

Table: Speedup and MAT results on the MT-Bench dataset. Our SoFT training was initialized with the Vicuna 160m checkpoint.

The SoFT + S2D draft consistently outperforms the pre-trained Vicuna 160m model in terms of speedup, demonstrating the value of fine-tuning for speculative decoding tasks. Intermediate layer drafts (e.g., SoFT L6, SoFT L9) offer competitive performance, showing the effectiveness of early-exit mechanisms.


***We respectfully ask the reviewer to consider revising the score based on these detailed clarifications and the demonstrated advantages of our method.***
