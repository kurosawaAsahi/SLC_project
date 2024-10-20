
Zhao, Z., Wu, L., Tang, C., Yin, D., Zhao, Y., & Luo, C. (2023). **Filler Word Detection with Hard Category Mining and Inter-Category Focal Loss**. *Microsoft Research Asia, Beijing, China; University of Science and Technology of China, Hefei, China*.
# what is filler and how to detect filler ?
Those who do not have professional training in speaking tend to use filler words such as “um” or “uh” frequently.
# Summary of Existing Filler Word Detection Methods
## 1. ASR-based Detection Methods
- use ASR models to transcribe speech first and then detect filler words from the transcript.
- ASR models often have difficulty transcribing filler words since they are trained on corpora with minimal filler content.
- Even when retrained with filler word datasets, ASR models do not outperform specialized filler word detection models and incur high computational costs.

## 2. ASR-free Detection Methods
- **Convolutional Recurrent Neural Network (CRNN)** or **Voice Activity Detection (VAD)**
- ASR-free methods are used to detect disfluencies, social signals, or other sound events (e.g., filler words, laughter, long silences, repetitions) without the use of ASR.
- These methods detect multiple events simultaneously, requiring more complex designs and datasets with comprehensive labels.

## A profound challenge
Words like "a", "the", or "umbrella" often get misclassified as filler words such as "uh" or "um."Traditional methods lump these confusing words into an "unknown" category

