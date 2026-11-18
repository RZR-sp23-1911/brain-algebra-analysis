# Brain Algebra — paper analysis

Course project (Master in Artificial Intelligence). Our team analyses, reproduces in part, and critically reviews the paper:

> Ferrante, M., Boccato, T., Toschi, N., & VanRullen, R. (2025). **Evidence for compositionality in fMRI visual representations via Brain Algebra.** *Communications Biology* 8:1263. https://doi.org/10.1038/s42003-025-08706-4

- Paper (open access): https://www.nature.com/articles/s42003-025-08706-4
- Authors' code: https://github.com/matteoferrante/BrainAlgebra
- Decoder used by the authors (Brain-Diffuser): https://github.com/ozcelikfu/brain-diffuser
- Data (Natural Scenes Dataset): https://naturalscenesdataset.org/

## The paper in brief

**Question.** Word embeddings can be combined with arithmetic ("king" − "man" + "woman" ≈ "queen"). Do visual representations in the human brain combine in a similar, algebraic way?

**Method.** The authors collect no new data. They reuse the Natural Scenes Dataset (NSD), where people viewed photographs in a 7T fMRI scanner.

1. **Base pattern.** The fMRI response of a subject to one test image (a vector with one value per voxel of visual cortex).
2. **Concept pattern.** For each of 12 concepts (man, woman, indoor, outdoor, summer, winter, day, night, crowded, empty, happy, sad), CLIP selects the 100 training images most related to the concept. The fMRI responses to those images are averaged.
3. **Thresholding.** Only the strongest voxels of the concept pattern are kept (from all voxels down to the top 5%).
4. **Brain algebra.** `z' = z_base + α · z_concept`, with α from 1 to 4.
5. **Decoding.** The new pattern is turned into an image with a pretrained fMRI-to-image decoder (Brain-Diffuser), trained separately for each of four subjects.
6. **Evaluation.** CLIP similarity between the decoded image and (a) the concept and (b) the original image, compared with random baselines.

**Main result.** Adding a concept pattern moves the decoded image towards the concept while keeping part of the original content (for example, a skater plus "winter" becomes a snowboarder on snow). The effect is present even when only a small fraction of voxels is modified.

**Limits stated by the authors.** The evaluation depends on the decoder; concepts need enough training examples; concepts are not orthogonal (for example, "happy" tends to add food and "crowded" adds people); no statistical hypothesis tests are reported.

## Team and roles

Everyone reads the whole paper; each member is responsible for one part.

| Role | Responsibility | Member |
|---|---|---|
| Problem framer (PF) | The question, the context and why it matters | *to be added* |
| Methodology expert (ME) | The method in detail | *to be added* |
| Coder (CO) | Running the authors' code, demo | *to be added* |
| Data scientist (DS) | Dataset description and exploratory data analysis | *to be added* |
| Results interpreter (R) | Reading and explaining the results | *to be added* |
| Challenger (CH) | Limitations and criticism | *to be added* |

## What is in this repository

| Path | Content |
|---|---|
| `README.md` | This overview of the paper and of our work |
| `notebooks/NSD_EDA.ipynb` | Exploratory data analysis of NSD, in three parts, with outputs saved |
| `coco_annotations/nsd_captions.csv` | COCO captions of the 73,000 NSD images (365,193 rows) |
| `coco_annotations/nsd_instances.csv` | COCO object annotations of the NSD images (516,477 rows) |
| `requirements.txt` | Python packages needed by the notebook |

The two CSV files are extracted from the COCO 2017 annotations, which are published by the COCO Consortium under a Creative Commons Attribution 4.0 License.

**Not in the repository:** the NSD files (see below) and the Python environment.

## The data

### Natural Scenes Dataset (NSD)

Allen et al. (2022), *Nature Neuroscience*. Eight participants viewed 73,000 photographs from COCO in a 7T scanner, over 30–40 sessions each. Every participant saw 10,000 images (9,000 unique to them and 1,000 shared by all), up to three times.

The paper uses four subjects (1, 2, 5, 7), the `nsdgeneral` mask of visual cortex, and single-trial betas at 1.8 mm resolution.

NSD is free to use, but each user must first accept the **NSD Data Access Agreement** linked from https://naturalscenesdataset.org/. For this reason, and because the beta files are far above GitHub's 100 MB limit per file, the NSD files are not stored here. They are in a public S3 bucket and can be downloaded without an AWS account.

Files used by the notebook (subject 1), to be placed in the project root as shown:

| File in the bucket `s3://natural-scenes-dataset/` | Local path | Size |
|---|---|---|
| `nsddata/experiments/nsd/nsd_stim_info_merged.csv` | `nsd_stim_info_merged.csv` | 11 MB |
| `nsddata/experiments/nsd/nsd_expdesign.mat` | `nsd_expdesign.mat` | 0.3 MB |
| `nsddata/ppdata/subj01/func1pt8mm/roi/nsdgeneral.nii.gz` | `nsdgeneral.nii.gz` | 0.1 MB |
| `nsddata_betas/ppdata/subj01/func1pt8mm/betas_fithrf_GLMdenoise_RR/betas_session01.nii.gz` … `betas_session05.nii.gz` | `betas/subj01/` | 5 × 490 MB |
| `…/betas_fithrf_GLMdenoise_RR/ncsnr.nii.gz` | `betas/subj01/` | 1 MB |
| `…/betas_fithrf_GLMdenoise_RR/R2.nii.gz` | `betas/subj01/` | 2.4 MB |

Example (one line per command):

```
pip install awscli
python -m awscli s3 cp --no-sign-request s3://natural-scenes-dataset/nsddata/experiments/nsd/nsd_stim_info_merged.csv .
python -m awscli s3 cp --no-sign-request s3://natural-scenes-dataset/nsddata/ppdata/subj01/func1pt8mm/roi/nsdgeneral.nii.gz .
python -m awscli s3 cp --no-sign-request s3://natural-scenes-dataset/nsddata_betas/ppdata/subj01/func1pt8mm/betas_fithrf_GLMdenoise_RR/betas_session01.nii.gz betas/subj01/
```

Do not download the whole bucket: it is several terabytes.

### COCO annotations

Captions and object annotations for the NSD images come from COCO 2017. The two tables the notebook needs are already in `coco_annotations/`, so nothing has to be downloaded for this part.

To rebuild them from the source, delete the two CSV files and save the original archive (241 MB) as `coco_annotations/annotations_trainval2017.zip`; the notebook then reads it directly:

```
http://images.cocodataset.org/annotations/annotations_trainval2017.zip
```

## Running the EDA notebook

```
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
jupyter lab notebooks/NSD_EDA.ipynb
```

The notebook is saved with its outputs, so tables and charts can be read directly on GitHub without running anything. Every chart is stored twice: as an interactive plotly figure (shown in Jupyter and VS Code) and as a static image (shown on GitHub). Creating the static images needs the `kaleido` package and a local Chrome or Chromium.

To run it:

- Parts 1 and 2 need one NSD file, `nsd_stim_info_merged.csv` (11 MB), next to the COCO tables that are already here.
- Part 3 also needs the mask, five beta sessions and the two quality maps (about 2.5 GB to download), and about 4 GB of free RAM on the first run. The first run saves the masked betas as `betas/subj01/X_nsdgeneral_sessions01-05.npy`, so later runs are fast.

## Exploratory data analysis: what we found

All numbers are for subject 1 unless stated otherwise. Part 3 uses a subsample: sessions 1–5 (3,750 of 27,750 trials).

**Part 1 — Experiment design**

- 73,000 images; each subject has 9,000 unique images and 1,000 shared ones.
- With the first 37 of 40 sessions we reproduce exactly the numbers in the paper: 8,859 training images (24,980 trials) and 982 test images (2,770 trials). Subjects 1, 2, 5 and 7 have identical counts.
- 84% of the images have all three repetitions; the rest have two, one or none.
- Repetitions of an image are far apart: the median distance between the first and the third viewing is 10 sessions.

**Part 2 — Image content (COCO)**

- Captions are short and uniform: 10.4 words on average (standard deviation 2.4), about five per image.
- Half of the images (50.7%) contain a person; every other object category is in fewer than 10%.
- Abstract concepts are almost absent from captions: "summer" and "sad" appear for 2–3 images. With a strict criterion (at least 3 of 5 captions), only "man", "woman" and "winter" reach 100 training images. This is why the authors need CLIP on the images rather than a text search.
- Some concepts co-occur with people far above the 50.7% average: "crowded" 93%, "happy" 91%, "winter" 85%. A concept pattern built from such images carries "people" as well as the concept.
- The test set is slightly simpler than the training set: 6.2 against 7.1 objects per image, and 43.8% against 51.5% images with people. A resampling check shows this is not chance.

**Part 3 — fMRI data**

- The mask keeps 15,724 voxels, about 15% of the brain of subject 1.
- A typical response is about 1% signal change, with a standard deviation of 2.6%.
- Single trials are noisy: for a typical voxel, 11.5% of the variance of one trial is due to the image; averaging three repetitions raises this to 28%.
- There is a measurable session effect: a typical voxel shifts by 0.16 standard deviations in a session, about seven times more than in random groups of trials. The per-voxel standardization used by Brain-Diffuser does not remove it.
- The data clearly carries information about the image: two viewings of the same image correlate at about 0.15, different images at 0. The same image has the higher correlation in 87–90% of comparisons.

## The paper compared with the code

Points we checked against the published paper and the public code:

- **Number of sessions.** The paper does not state it. Brain-Diffuser's `prepare_nsddata.py` loads 37 sessions, and 37 sessions reproduce the paper's image and trial counts.
- **Concept selection.** The paper describes it as the top 100 images by CLIP cosine similarity. In `BrainAlgebra_SingleConcept.ipynb` the image and text embeddings are multiplied without normalization, a softmax is applied across the two concepts of a pair, and the top 100 images by that probability are taken. A concept is therefore defined in contrast to its opposite.
- **Concept list.** The code contains ten pairs (including cat/dog, still/moving, left/right, top/down); the paper reports six. The text prompts are "indoor scene" and "outdoor scene".
- **Normalization.** Brain-Diffuser divides betas by 300 and standardizes each voxel over the whole training set, not per session.
- **Evaluation set.** The quantitative evaluation uses a fixed random subset of 100 test images, not all 982.

We read the current version of the authors' notebook; it may differ from the version used for the paper.

## Reduced test of Brain-Diffuser (stage 1)

We ran the first stage of the decoder (VDVAE) on limited hardware (RTX 5070, 12 GB VRAM, 32 GB RAM) with our own reduced scripts, using five sessions of subject 1 (2,267 training and 225 test images). The scripts are not part of this repository yet.

- Two-way identification on pixels: 79% (chance is 50%); 2.4 GB of VRAM used.
- The original scripts do not run as they are on this hardware: `prepare_nsddata.py` needs more than 130 GB of RAM, and `vdvae_regression.py` is missing an import.
- The `environment.yml` of Brain-Diffuser pins an old PyTorch that does not support this GPU; a recent PyTorch build is required.
- Stage 2 (Versatile Diffusion) does not fit in 12 GB and was not run.

## Status

- [x] Paper read and explained; authors' code inspected
- [x] Dataset access and download of a subsample
- [x] Exploratory data analysis notebook (DS)
- [x] Reduced Brain-Diffuser stage 1 test
- [ ] Contributions of the other roles
- [ ] Slides for the course presentation

## References

- Ferrante, M., Boccato, T., Toschi, N., & VanRullen, R. (2025). Evidence for compositionality in fMRI visual representations via Brain Algebra. *Communications Biology*, 8, 1263.
- Allen, E. J., et al. (2022). A massive 7T fMRI dataset to bridge cognitive neuroscience and artificial intelligence. *Nature Neuroscience*, 25, 116–126.
- Ozcelik, F., & VanRullen, R. (2023). Natural scene reconstruction from fMRI signals using generative latent diffusion. *Scientific Reports*, 13, 15666.
- Lin, T.-Y., et al. (2014). Microsoft COCO: Common Objects in Context. *ECCV 2014*.
- Radford, A., et al. (2021). Learning transferable visual models from natural language supervision. *ICML 2021*.
