# Recent Tasks
## MPhil Application
preparing the materials(completed)

## Direct Speech-to-Speech Translation with Discrete Units

I initially attempted to use unitY but faced issues running it successfully. 
I have now decided to revert to using Direct Speech-to-Speech Translation with Discrete Units for my project. 
a helpful implementation video, which can be found [here](https://www.youtube.com/watch?v=HIAt9kawqsQ&list=PLvELbYeZ7GEFsYxurUXIXpmksCUz6-Z5M&index=6).<br>
### data preparation
audio file preparation:ogg2wav,divide into test, train, dev(200000,25000,25000),disk quota exceed(wav much larger than ogg),the same pairs change to same file names<br>
###  prepare target discrete units
Quantize using a pretrained  K-means clustering model. <br>
decode units from speech（use 100 units from the sixth layer (--layer 6) of the HuBERT Base model):HUBERT-BASE+KM100
1. we have to prepare a separated_manifest_of_audio_files_to_quantize(a file with paths and length of input audio files)
2. Quantize using the learned clusters( we use a pretrained HuBERT Base + KM100 model) (configuring the environment, ensuring compatibility with dependencies, and resolving issues with data processing takes time)
### format the data

   


## a new pipeline for data filtering
### acoustic noise(how to filter background from the audio)
detection of human speech by targeting a specific frequency range (300 Hz to 1500 Hz), isolating speech in complex audio data. paper ["Voice Activity Detection (VAD) in Noisy Environments"](https://arxiv.org/html/2312.05815v1)
### speech noise(Filler Word Detection)
 detecting and removing filler words such as "uh" and "um" from speech recordings. paper ["Filler Word Detection with Hard Category Mining and Inter-Category Focal Loss"](https://arxiv.org/pdf/2304.05922).
- Use of **focal loss** to focus on difficult examples.
- **Auxiliary categories** to separate confusing non-filler words.
### cope with mistranslation (original plan)
### Summary of the Approach

- **Speech Encoder**: The acoustic features \(X_s\) from speech are processed by a **speech encoder** to generate robust linguistic representations \(Z_s\), removing speaker traits, emotions, and noise. This process is defined as \(Z_s = F_{se}(X_s)\). The paper focuses on using mHuBERT and Whisper for speech encoding.

- **Adaptor**: The **adaptor** serves as a bridge between the speech encoder and the **Large Language Model (LLM)**, aligning speech features \(Z_s\) with the LLM's embedding space. The transformation is represented by \(H_s = F_{ada}(Z_s)\), where a 3-layer multilayer perceptron (MLP) is used to project the speech features into the LLM's input realm.

This structure enables smooth integration of speech data into a text-based LLM for various natural language tasks.





