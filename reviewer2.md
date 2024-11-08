**1. Target-Dependent Baselines on Larger Models**

Thank you for pointing out the need for additional comparisons with target-dependent baselines on larger models. We aimed to provide a fair comparison in Table 1 by focusing on results from target-dependent approaches, such as Eagle, on a single target model (Vicuna 7B). This was because Eagle’s draft model, trained specifically for Vicuna 7B, cannot generalize to other target models like Vicuna 13B or LLaMA2 Chat 70B due to its inherent target dependence.

To address your concern, we conducted additional experiments with Eagle, training its draft model specifically for LLaMA2 Chat 70B and using it as the target model in greedy decoding mode (T=0). The table below summarizes the results, showing performance under of Eagle on a large-scale model, LLaMA2 Chat 70B. 

| **Method**                   | **LLaMA Chat 70B** (T=0) |               |
|------------------------------|--------------------------|---------------|
|                              | **Speedup**              | **MAT**       |
| Eagle (No Attention Tree)    | 2.21×                   | 2.38          |
| SFT + SD                 | 1.94×                   | 2.46          |
| SFT + EarlyExit L6 + SD      | 1.32×                   | 1.46          |
| SFT + EarlyExit L9 + SD      | 1.54×                   | 1.80          |
| SoFT L6 + SD                 | 1.83×                   | 2.05          |
| SoFT L9 + SD                 | 1.92×                   | 2.27          |
| SoFT L12 + SD                | 1.91×                   | 2.39          |
| SoFT + S2D (ours)            | 1.95×               | 2.36          |


**2. Motivation for Using a Single Draft Model**

We acknowledge the reviewer’s observation that deploying multiple draft models may seem feasible due to their relatively low cost compared to large target models. However, our approach of utilizing a single draft model provides distinct and critical advantages that address real-world scalability and efficiency challenges.

***Reduced Training Cost:***
Training a separate draft model for each target model incurs significant memory and computational overhead. This is particularly true for target-dependent approaches like Medusa, which require the full target model to remain in memory during training. As shown in the table below, Medusa's memory consumption grows with the size of the target model, eventually exceeding hardware limits for larger models (e.g., LLaMA2 Chat 70B). In contrast, our SoFT draft model is target-agnostic and needs to be trained only once, making it far more efficient in terms of training memory and cost.

| **Method/Target Model**    | **Vicuna 7b** | **Vicuna 13b** | **LLaMA2 Chat 70b** |
|----------------------------|---------------|----------------|---------------------|
| SoFT Draft (12 Layers)     | 92,744 MB     | 92,744 MB      | 92,744 MB           |
| Medusa                     | 113,416 MB    | 152,616 MB     | OOM                 |

**Table:** Memory consumption during training for SoFT Draft vs. Medusa across target models of varying sizes. Note: SoFT Draft requires training only once, while Medusa must be retrained for each target model.

***Scalability in Deployment:***
A single draft model simplifies deployment pipelines in multi-target scenarios, especially for organizations managing diverse environments with varying resource and latency requirements (e.g. cloud service providers such as AWS). By serving multiple targets without retraining, our approach reduces operational complexity and storage costs. This is particularly advantageous in:

- Edge AI: Smaller models (e.g., Vicuna 7B) can be deployed on resource-constrained devices, while larger models (e.g., LLaMA2 Chat 70B) can be used in high-performance cloud setups.
- Dynamic and Adaptive Workflows: A single draft model supports seamless transitions between target models based on task complexity, resource availability, or deployment constraints.

***Alignment with Emerging Trends:***
The trend toward many-in-one model frameworks, such as FLEXTRON [ICML 2024], underscores the growing need for adaptable solutions capable of supporting multiple targets. Our SoFT + S2D approach anticipates and addresses this demand, providing an efficient and scalable alternative to traditional target-dependent methods.


**3. Comparison with Target-Dependent Approaches and Integration of Attention Tree**

As highlighted earlier, Eagle is a target-dependent approach that requires training drafts specific to each target model, significantly increasing memory and time costs during the training phase. This dependency becomes particularly burdensome when scaling to multiple target models, as each draft must be trained anew based on the target's outputs. In contrast, our SoFT draft model is target-agnostic, requiring a single training phase to serve multiple target models, thus reducing overall training complexity and deployment costs.

***Integration of Attention Tree in S2D:***
One key technique used by target-dependent methods such as Eagle to improve Mean Accepted Tokens (MAT) is the Attention Tree. To ensure a fair comparison and further enhance our S2D approach, we integrated the Attention Tree mechanism into the draft generation process. This enhancement demonstrated notable improvements in MAT and speedup, particularly for Vicuna 7B and 13B, across both greedy (T=0) and non-greedy (T=1) setups.

The integration of Attention Tree allowed us to achieve up to a 2× speedup on Vicuna 7B in greedy decoding. These results indicate that our S2D approach, when paired with techniques like Attention Tree, can effectively compete with target-dependent methods while maintaining the flexibility of serving multiple target models.

***Experimental Comparison:***
The table below summarizes the performance of various methods on the MT-Bench dataset in terms of speedup and MAT for multi-target inference acceleration:

| **Method**                  | **Vicuna 7B (T=0)** |                 | **Vicuna 13B (T=0)** |                 | **Vicuna 7B (T=1)** |                 | **Vicuna 13B (T=1)** |                 |
|-----------------------------|---------------------|-----------------|----------------------|-----------------|---------------------|-----------------|----------------------|-----------------|
|                             | **Speedup**         | **MAT**         | **Speedup**          | **MAT**         | **Speedup**         | **MAT**         | **Speedup**          | **MAT**         |
| **Eagle [1]**               | 2.62×              | 3.84            | ✖️                   | ✖️              | 2.05×              | 3.42            | ✖️                   | ✖️              |
| Eagle (No Attention Tree)   | 2.04×              | 3.11            | ✖️                   | ✖️              | 1.72×              | 2.73            | ✖️                   | ✖️              |
| **Medusa [2]**              | 1.74×              | 2.51            | ✖️                   | ✖️              | 1.93×              | 2.80            | ✖️                   | ✖️              |
| Medusa (No Attention Tree)  | 1.34×              | 1.76            | ✖️                   | ✖️              | 1.48×              | 1.92            | ✖️                   | ✖️              |
| **Hydra [3]**               | 2.14×              | 2.70            | ✖️                   | ✖️              | 2.36×              | 4.01            | ✖️                   | ✖️              |
| Hydra (No Attention Tree)   | 1.78×              | 2.65            | ✖️                   | ✖️              | 2.03×              | 3.11            | ✖️                   | ✖️              |
| SFT + SD                    | 1.19×              | 3.19            | 1.21×               | 3.05            | 1.16×              | 3.44            | 1.10×               | 3.16            |
| **SFT + SD (Attention Tree)**          | 1.68×              | 4.68            | 1.64×               | 4.59            | 1.33×              | 4.16            | 1.54×               | 4.15            |
| SFT + EarlyExit L6 + SD     | 0.98×              | 1.51            | 1.00×               | 1.51            | 0.96×              | 1.52            | 1.03×               | 1.51            |
| **SFT + EarlyExit L6 + SD (Attention Tree)** | 1.19×         | 2.51            | 1.18×               | 2.52            | 1.20×              | 2.41            | 1.15×               | 2.41            |
| SFT + EarlyExit L9 + SD     | 1.08×              | 1.99            | 1.05×               | 1.98            | 0.99×              | 2.04             | 1.12×               | 2.04            |
| **SFT + EarlyExit L9 + SD (Attention Tree)** | 1.36×              | 3.44            | 1.43×               | 3.43            | 1.11×              | 3.17            | 1.23×               | 3.17            |
| SoFT L6 + SD                | 1.38×              | 2.43            | 1.38×               | 2.40            | 1.30×              | 2.53            | 1.35×               | 2.87            |
| **SoFT L6 + SD (Attention Tree)**      | 1.99×              | 4.08            | 1.95×               | 4.08            | 1.86×              | 3.73            | 1.74×               | 3.70            |
| SoFT L9 + SD                | 1.16×              | 3.00            | 1.31×               | 2.78            | 1.26×              | 3.05           | 1.32×               | 2.87            |
| **SoFT L9 + SD (Attention Tree)**      | 1.84×              | 4.45            | 1.79×               | 4.42            | 1.52×              | 3.89            | 1.68×               | 4.01            |
| SoFT L12 + SD                | 1.17×              | 3.11           | 1.23×               | 3.01            | 1.07×              | 3.22           | 1.20×               | 3.17            |
| **SoFT L12 + SD (Attention Tree)**      | 1.63×              | 4.61            | 1.62×               | 4.59            | 1.44×              | 4.12            | 1.58×               | 4.23            |
| SoFT + S2D (Ours)           | 1.34×              | 2.86            | 1.38×               | 2.76            | 1.27×              | 3.01            | 1.38×               | 2.89            |
| **SoFT + S2D (Attention Tree)**        | 2.00×              | 4.11            | 1.93×               | 4.11            | 1.84x                 | 3.77              | 1.77x                  | 3.79              |


***We kindly ask the reviewer to reconsider their score based on the clarifications and additional evidence provided, which highlight the scalability, practicality, and robust performance of our method.***
