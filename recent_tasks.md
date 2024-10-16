# Recent Tasks
## MPhil Application
preparing the materials(completed)

## Direct Speech-to-Speech Translation with Discrete Units

I initially attempted to use unitY but faced issues running it successfully. 
I have now decided to revert to using Direct Speech-to-Speech Translation with Discrete Units for my project. 
a helpful implementation video, which can be found [here](https://www.youtube.com/watch?v=HIAt9kawqsQ&list=PLvELbYeZ7GEFsYxurUXIXpmksCUz6-Z5M&index=6).
audio file preparation(completed)

## a new pipeline for data filtering
### acoustic noise(how to filter background from the audio)
detection of human speech by targeting a specific frequency range (300 Hz to 1500 Hz), isolating speech in complex audio data. paper ["Voice Activity Detection (VAD) in Noisy Environments"](https://arxiv.org/html/2312.05815v1)
### speech noise(Filler Word Detection)
 detecting and removing filler words such as "uh" and "um" from speech recordings. paper ["Filler Word Detection with Hard Category Mining and Inter-Category Focal Loss"](https://arxiv.org/pdf/2304.05922).
- Use of **focal loss** to focus on difficult examples.
- **Auxiliary categories** to separate confusing non-filler words.
### cope with mistranslation (original plan)




