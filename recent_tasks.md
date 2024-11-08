# Recent Tasks
## MPhil Application
preparing the materials(completed)

## Direct Speech-to-Speech Translation with Discrete Units

I initially attempted to use unitY but faced issues running it successfully. 
I have now decided to revert to using Direct Speech-to-Speech Translation with Discrete Units for my project. 
a helpful implementation video, which can be found [here](https://www.youtube.com/watch?v=HIAt9kawqsQ&list=PLvELbYeZ7GEFsYxurUXIXpmksCUz6-Z5M&index=6).<br>
### data preparation
#### Audio Specifications:
- **Sample Width**: 4 bytes (32-bit audio)
- **Sample Rate**: 16000 Hz
- **Duration**: 10 seconds
- **Channels**: Mono
#### Original WAV File Size Calculation:
16000 samples/second × 10 seconds × 4 bytes/sample = 640,000 bytes (640 KB)
#### Solution to Quota Problem:
When reading OGG files, the `sample_width` is reduced to 2 (instead of 4 in the original file) and saved as WAV. This reduces the WAV file size by about half. However, even after this reduction, 1.4 million pairs still wouldn't fit within the quota, so I saved 250,000 pairs instead.
**Note**: Lowering audio quality may negatively impact the model's performance.
#### Audio File Preparation: OGG to WAV Conversion and Dataset Splitting
- **Target Split**: 
  - Training: 200,000 pairs
  - Testing: 25,000 pairs
  - Development: 25,000 pairs
- **Disk Quota Issue**: 
  - WAV files are significantly larger than OGG files, causing a disk quota exceedance. The same audio pairs are renamed to ensure consistency between the file names.

###  prepare target discrete units
Quantize using a pretrained  K-means clustering model. <br>
decode units from speech（use 100 units from the sixth layer (--layer 6) of the HuBERT Base model):HUBERT-BASE+KM100
1. we have to prepare a separated_manifest_of_audio_files_to_quantize(a file with paths and length of input audio files)
2. Quantize using the learned clusters( we use a pretrained HuBERT Base + KM100 model) (configuring the environment, ensuring compatibility with dependencies, and resolving issues with data processing takes time)
### format the data
#### S2UT data
(completed, to run the code I have to modify fairseq)
#### Multitask data
Actually this step is optional.Better performance
### Training without mulititask
100 discrete units as target:
#### Model Architecture:
- `--arch s2ut_transformer_fisher`: a Transformer-based architecture（pretrained on Fisher）
- `--share-decoder-input-output-embed`: Shares weights between the decoder input embeddings and output embeddings, reducing model size.
#### Training Hyperparameters:
- `--label-smoothing 0.2`:prevent overfitting
- `--dropout 0.1`: prevent overfitting
- `--attention-dropout 0.1`: randomly setting 0.1 of attention weight to zero.
- `--relu-dropout 0.1`: randomly setting 0.1 of the neurons' outputs after the ReLU activation to zero during training
#### Learning Rate and Optimization:
- `--lr 0.0005`: learning rate
- `--lr-scheduler inverse_sqrt`:decreases the learning rate according to the square root of the update step after the warmup phase.
- `--warmup-init-lr 1e-7`: initial learning rate during the warmup phase.
- `--warmup-updates 10000`: The first 10,000 updates, the learning rate will start very small (1e-7) and gradually increase to the base learning rate (0.0005)
- `--optimizer adam`: 
- `--adam-betas "(0.9, 0.98)"`: momentum and variance for the moving averages of gradient updates.
- `--clip-norm 10.0`:to prevent exploding gradients.

- **Number of GPUs**: 4 x RTX 2080
- **Total Epochs**: 63
  #### Validation Results on the Dev Set while training
Unfortunately, forget to use tensorboard while training
The table below summarizes the validation results from the dev set for the last three epochs:

| Epoch | Loss  | NLL Loss | Perplexity (PPL) | Words per Second (WPS) | Words per Batch (WPB) | Batch Size (BSZ) | Number of Updates | Best Loss |
|-------|-------|----------|------------------|------------------------|-----------------------|------------------|-------------------|-----------|
| 61    | 3.882 | 2.682    | 6.42             | 87315.5                 | 4089.8                | 20.4             | 149,056            | 3.882     |
| 62    | 3.881 | 2.681    | 6.41             | 88072.3                 | 4089.8                | 20.4             | 151,499            | 3.881     |
| 63    | 3.879 | 2.676    | 6.39             | 86442.2                 | 4089.8                | 20.4             | 153,942            | 3.879     |
### INFERENCE and EVALUATION
with beam search (beam size = 10):
#### BLEU Score
- **BLEU4**: 8.64
#### Detailed BLEU Components
- **N-gram Precision**:
  - **1-gram Precision**: 30.6% 
  - **2-gram Precision**: 12.7% 
  - **3-gram Precision**: 5.6% 
  - **4-gram Precision**: 2.6%
  - **Brevity Penalty**: 1.000
    - **Ratio**: 1.690 – the generated output is 69% longer than the reference
### Training with mulititask
#### multi-task data preparation
1. **Transcription and Tokenization**:
   - Using the Whisper model to transcribe audio files into text for both `source` and `target` tasks.
   - Tokenize the transcribed text, splitting it into individual tokens (characters).

2. **Word Frequency Calculation**:
   - Calculate the frequency of each token in the transcriptions to generate a frequency dictionary, which is useful for balanced training.

#### Multi-Task Learning Configuration
1. **source_letter**
   - **Decoder Type**: Transformer
   - **Dictionary Path**: `${DATA_ROOT}/source_letter/dict.txt`
   - **Data Path**: `${DATA_ROOT}/source_letter`
   - **Encoder Layers**: 6
   - **Loss Weight**: 8.0
   The `source_letter` task is configured with a 6-layer encoder and utilizes a Transformer decoder. The dictionary（ store word frequencies）, a loss weight of 8.0.

2. **target_letter**
   - **Decoder Type**: Transformer
   - **Dictionary Path**: `${DATA_ROOT}/target_letter/dict.txt`
   - **Data Path**: `${DATA_ROOT}/target_letter`
   - **Encoder Layers**: 8
   - **Loss Weight**: 8.0
#### Training Hyperparameters
The training hyperparameters for this multi-task setup are kept consistent with non-multi-task configurations, with one exception: the learning rate. 
While we initially set the learning rate to the recommended value of 0.0005, as per the reference paper, this caused frequent gradient explosion issues, leading to unstable training that could not be sustained
To address this, the learning rate was reduced to **0.00001**.
At the **90th epoch**, we increase the learning rate back to **0.0005** to speed up convergence. By this stage, the model has likely stabilized, and a higher learning rate can accelerate final convergence.<br>
**Number of GPUs**: 4 x RTX 2080<br>
**Total Epochs**: 130








## a new pipeline for data filtering
### acoustic noise(how to filter background from the audio)
detection of human speech by targeting a specific frequency range (300 Hz to 1500 Hz), isolating speech in complex audio data. paper ["Voice Activity Detection (VAD) in Noisy Environments"](https://arxiv.org/html/2312.05815v1)
### speech noise(Filler Word Detection)
 detecting and removing filler words such as "uh" and "um" from speech recordings. paper ["Filler Word Detection with Hard Category Mining and Inter-Category Focal Loss"](https://arxiv.org/pdf/2304.05922).
- Use of **focal loss** to focus on difficult examples.
- **Auxiliary categories** to separate confusing non-filler words.

Maybe this one would be better: paper ["TRANSCRIPTION FREE FILLER WORD DETECTION WITH NEURAL SEMI-CRFS"](https://arxiv.org/pdf/2303.06475).
### cope with mistranslation (original plan)
### Summary of the Approach

- **Speech Encoder**: The acoustic features \(X_s\) from speech are processed by a **speech encoder** to generate robust linguistic representations \(Z_s\), removing speaker traits, emotions, and noise. This process is defined as \(Z_s = F_{se}(X_s)\). The paper focuses on using mHuBERT and Whisper for speech encoding.

- **Adaptor**: The **adaptor** serves as a bridge between the speech encoder and the **Large Language Model (LLM)**, aligning speech features \(Z_s\) with the LLM's embedding space. The transformation is represented by \(H_s = F_{ada}(Z_s)\), where a 3-layer multilayer perceptron (MLP) is used to project the speech features into the LLM's input realm.

This structure enables smooth integration of speech data into a text-based LLM for various natural language tasks.





