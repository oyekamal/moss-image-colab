# MOSS-Image on a free Google Colab T4

Run [MOSS-Image](https://github.com/OpenMOSS/MOSS-Image), a brand-new 5B-parameter text-to-image model that is very good at writing text inside pictures, on Colab's **free T4**. No paid GPU, no API key, no ComfyUI.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/oyekamal/moss-image-colab/blob/main/notebooks/moss_image_t4.ipynb)

The official weights are 27 GB, which does not fit a free T4 (16 GB of GPU memory, 12 GB of RAM). This notebook stores the image model as **int8** and the text encoder as **4-bit** so that it does fit, with the model's own code unchanged.

![Marko waving, made at 1024x1024](examples/marko_wave_1024.png)

## Quick start

1. Open the notebook with the badge above and choose **Runtime → Change runtime type → T4 GPU**.
2. Run the cells from the top. The first run downloads about 25 GB (around 15 minutes) and loading takes about 2.5 minutes.
3. Edit the prompt in the last cell and run it. A 768×768 picture takes about 2 minutes.

The model files live on Colab's disk for the session only. A new session downloads them again, because a free Drive (15 GB) is too small for a 27 GB model. See *Model storage* below.

## What it makes

All pictures below were made on a free T4 with seed 42 (full prompts in [`examples/prompts.md`](examples/prompts.md)). The Marko wave, the English sign and the classroom are the direct output of the notebook's cells. The 1024×1024 picture at the top and the Urdu sign came from my earlier test scripts with the same settings.

| | |
|---|---|
| ![Marko waving](examples/marko_wave_768.png) | ![Marko with an English sign](examples/marko_sign_english.png) |
| Marko, the markhor mascot of the [Urdu Reading Course](https://github.com/oyekamal/urdu-reading-course), from a text description (768×768) | English text on a sign: **READ URDU WITH MARKO**, spelled correctly |
| ![A classroom](examples/classroom_welcome.png) | ![Marko with an Urdu sign](examples/marko_urdu_sign.png) |
| A village classroom whose chalkboard says **WELCOME** | An Urdu sign. The first word, **اردو**, is right; the second word is not. See the limits below. |

## Numbers

Measured on a free Colab T4 (Python 3.13, torch 2.11 with CUDA 13, diffusers 0.41, transformers 5.20):

| | Time | GPU memory |
|---|---|---|
| Download (first time) | ~14 min for 25 GB | |
| Load with quantization | ~2.5 min | 11.5 GB |
| 768×768, 30 steps | ~2 min | 12.9 GB peak |
| 1024×1024, 30 steps | ~3 min 46 s | 14.0 GB peak (tight) |

For comparison, the model card's own example uses 50 steps and bfloat16 on a large GPU. On a T4 the cost is time: a picture takes minutes, not seconds.

## Why quantization, and why this particular recipe

- **Image model in int8 (`optimum-quanto`), text encoder in 4-bit (`bitsandbytes`).** The text encoder is a large language model (about 17 GB in bfloat16), so it gets the harsher 4-bit squeeze. The image model is smaller (10 GB) and more sensitive, so it gets int8.
- **Why not 4-bit for the image model too?** I tried it first. The model's own code reads `layer.weight.dtype` to decide what type to compute in, and a 4-bit `bitsandbytes` layer stores its weights as packed bytes, so the code crashed with `expected scalar type Float but found Byte`. `optimum-quanto` weights report their normal dtype, so the unmodified code works. Patching those dtype lookups and removing CPU offload did not fix the 4-bit version, so I did not pursue it further.
- **float16, not bfloat16.** The T4 has no native bfloat16. Our test pictures came out clean in float16 (no NaNs, no black images).
- **`torchao` is uninstalled.** Colab ships a `torchao` older than `diffusers` 0.41 expects, which breaks `import diffusers` with `cannot import name 'FqnToConfig'`. Nothing here uses `torchao`.

## Model storage (Google Drive)

The notebook uses a small helper, `get_model()`, instead of a plain download:

1. It mounts Google Drive (Colab asks you to allow it) and keeps models in `MyDrive/colab-models/moss-image-sft`.
2. If a **complete** copy is already there, it is used as it is, with no download. A copy counts as complete only when a marker file was written after a finished download.
3. If it is missing, the helper asks Hugging Face how big the download is and compares that with your free Drive space. If it fits, it downloads straight into Drive. If it does not fit, it uses the session's disk and tells you.
4. An interrupted download is resumed, not restarted. Nothing is ever deleted.

At about 27 GB it **does not fit** in a free Google Drive (15 GB in total, shared with all your files), so on a free account it still downloads to the session's disk every time. With a Drive that has enough free space (for example Google One), the first run saves it and later sessions reuse it.

Say no to the Drive prompt, or set `USE_DRIVE = False` in the helper cell, to always use the session's disk.

**The Google Drive step is new and has not been run in Colab yet.** The model-download cell in this notebook was replaced by `get_model()` after the tested run described here. I tested that helper's logic on its own against a real Hugging Face repo with a stand-in Drive folder (five cases: downloads into Drive, reuses a finished copy without downloading, falls back to the local disk when Drive is too small, repairs an interrupted download, and the `USE_DRIVE = False` switch), and I checked that every notebook cell parses. I have not run the real Drive mount (it needs your click to allow access), nor the edited notebook end to end. If something fails, please open an issue with the error text.

## Limits and honest notes

- **Urdu text is only partly right.** The model draws convincing Urdu script, and the first word on the sign above is correct, but the second word is wrong. English text was spelled correctly in both of my tests (the sign and the chalkboard). If you need Urdu text, check each word.
- **Slow.** About 2 minutes per picture. The community [8-step distilled version](https://huggingface.co/lodestones/MOSS-Image-SFT-8-steps-dmd) should be faster, but I have not tried it here.
- **1024×1024 is near the memory limit** (14 of 15 GB). If it runs out of memory, use 768 or restart the runtime and run only the load and draw cells.
- **Only tested on flat art and one classroom scene, at 30 steps.** The model card's default is 50 steps, which I did not try. For photos or detailed scenes, try more steps.
- **No built-in content filter.** There is no safety checker in the model files or in the official pipeline. It is not an "uncensored" fine-tune either, and I only tested harmless prompts. You are responsible for what you generate.
- **Not tested:** other GPUs, other prompt types (tag lists, JSON layouts), LoRA loading, and the Colab web page click-through. Instead, I ran every cell of the notebook, in order, inside a real Colab T4 kernel through a script, and the pictures here are its output. If a cell fails for you, please open an issue with the error text.
- **The model is new.** Version pins here (`diffusers>=0.38`, `transformers>=5.3` come with Colab today) may need updating as things change.

## Files

| Path | What |
|---|---|
| `notebooks/moss_image_t4.ipynb` | The notebook: install, download, load, draw |
| `examples/` | The pictures above and `prompts.md` with the exact prompts |

Driving Colab from a terminal instead of the web page: see the scripts in [colab-3d-generation](https://github.com/oyekamal/colab-3d-generation/tree/main/tools).

## Credits and licences

- **MOSS-Image** is by [OpenMOSS](https://github.com/OpenMOSS/MOSS-Image) and released under Apache-2.0. The weights are **not** in this repo; the notebook downloads them from [Hugging Face](https://huggingface.co/OpenMOSS-Team/MOSS-Image-SFT).
- **This notebook, README and scripts:** MIT, see `LICENSE`.
- **Marko** is the mascot of the Urdu Reading Course by the repo owner. The example pictures of him are generated, and are shared as examples.
