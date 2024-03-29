# TGATE: Cross-Attention Makes Text-Conditional Diffusion Inference Cumbersome

## 🥳 Quick Introduction

> We find that cross-attention outputs converge to a fixed point during the initial denoising steps. Consequently, the entire inference process can be divided into two stages: an initial semantics-planning phase, during which the model relies on text to plan visual semantics, and a subsequent fidelity-improving phase, during which the model tries to generate images from previously planned semantics. Surprisingly, ignoring text conditions in the fidelity-improving stage not only reduces computation complexity, but also slightly increases model performance. This yields a simple and training-free method called TGATE for efficient generation, which caches the cross-attention output once it converges and keeps it fixed during the remaining inference steps. 

![](./src/visualization.png)
>  The images generated by the diffusion model with or without TGATE. Our method can accelerate the diffusion model without generation performance drops. It is training-free and can be widely complementary to the existing studies.

## 📄 Updates

* Code will release soon. 

* Technical Report will come soon.
  
* 2024/03/28: LCM (SD-XL) w/ TGATE is open source.
  
* 2024/03/28: TGATE is open source.

## 🚀 Major Features 

* Training-Free.
* Easily Integrate into Existing Frameworks.
* Only a few lines of code are required.
* Friendly support CNN-based U-Net, Transformer, and Consistency Model
  
## 📖 Key Observation

![](./src/observation.png)

>  The images generated by the diffusion model at different denoising steps. The first row feeds the text embedding to the cross-attention modules for all steps. The second row only uses the text embedding from the first step to the 10th step, and the third row inputs the text embedding from the 11th to the 25th step.

We summarize our observations as follows:

* Cross-attention converges early during the inference process, which can be characterized by a semantics-planning and a fidelity-improving stages. The impact of cross-attention is not uniform in these two stages.

* The semantics-planning embeds text through cross-attention to obtain visual semantics.

* The fidelity-improving stage improves the generation quality without the requirement of cross-attention. In fact, a null text embedding in this stage can improve performance.

## 🖊️ Method

* Step 1: TGATE caches the attention outcomes from the semantics-planning stage.
```
if gate_step == cur_step:
    hidden_uncond, hidden_pred_text = hidden_states.chunk(2)
    cache = (hidden_uncond + hidden_pred_text ) / 2
```

* Step 2: TGATE reuses them throughout the fidelity-improving stage.

```
if cross_attn and (gate_step<cur_step):
    hidden_states = cache
```

## 📄 Results
| Model                 | MACs     | Param     | Latency | Zero-shot 10K-FID on COCO |
|-----------------------|----------|-----------|---------|---------------------------|
| SD-1.5                | 16.938T  | 859.520M  | 7.032s  | 23.927                    |
| SD-1.5 w/ TGATE       | 9.875T   | 815.557M  | 4.313s  | 20.789                    |
| SD-2.1                | 38.041T  | 865.785M  | 16.121s | 22.609                    |
| SD-2.1 w/ TGATE       | 22.208T  | 815.433 M | 9.878s  | 19.940                    |
| SD-XL                 | 149.438T | 2.570B    | 53.187s | 24.628                    |
| SD-XL w/ TGATE        | 84.438T  | 2.024B    | 27.932s | 22.738                    |
| Pixart-Alpha          | 107.031T | 611.350M  | 61.502s | 38.669                    |
| Pixart-Alpha w/ TGATE | 70.225T  | 462.585M  | 40.648s | 35.726                    |
| LCM (SD-XL)           | 11.955T  | 2.570B    | 3.805s  | 25.044                    |
| LCM w/ TGATE          | 11.171T  | 2.024B    | 3.533s  | 24.595                    |

## 🛠️ Requirements 

* diffusers==0.27.0.dev0
* pytorch==2.2.0
* transformers


## 🌟 Usage

SD-1.5 w/ TGATE: generate an image with the caption: "A coral reef bustling with diverse marine life." 

```
cd SD_1_5
python generate.py
```


SD-2.1 w/ TGATE: generate an image with the caption: "High quality photo of an astronaut riding a horse in space" 

```
cd SD_2_1
python generate.py
```


SD-XL w/ TGATE: generate an image with the caption: "Astronaut in a jungle, cold color palette, muted colors, detailed, 8k" 

```
cd SDXL
python generate.py
```

Pixart-Alpha w/ TGATE: generate an image with the caption: "An alpaca made of colorful building blocks, cyberpunk." 

```
cd PixArt_alpha
python generate.py
```

LCM w/ TGATE: generate an image with the caption: "Self-portrait oil painting, a beautiful cyborg with golden hair, 8k" 

```
cd LCM
python generate.py
```




##  📖 Related works: 

We encourage the users to read [DeepCache](https://arxiv.org/pdf/2312.00858.pdf) and [Adaptive Guidance](https://arxiv.org/pdf/2312.12487.pdf)

| Methods           | U-Net   | Transformer | Consistency Model |
|-------------------|---------|-------------|-------------------|
| DeepCache         | &check; | &cross;     | &check;           |
| Adaptive Guidance | &check; | &check;     | &cross;           |
| TGATE (Ours)      | &check; | &check;     | &check;           |


Compared with DeepCache: 
* TGATE can cache one time and re-use the cached feature until ending sampling. 
* TGATE is more friendly for Transformer-based Architecture and mobile devices since it drops the high-resolution cross-attention.
* TGATE is complementary to DeepCache. 

Compared with Adaptive Guidance: 
* TGATE can reduce the parameters in the second stage. 
* TGATE can further improve the inference efficiency.
* TGATE is complementary to non-cfg framework, e.g. latent consistency model. 
* TGATE is open source.

## Acknowledgment 

* We thank [prompt to prompt](https://github.com/google/prompt-to-prompt)  and [diffusers](https://huggingface.co/docs/diffusers/index) for the great code. 

* The computational complexity are calculated based on [calflops](https://github.com/MrYxJ/calculate-flops.pytorch).
