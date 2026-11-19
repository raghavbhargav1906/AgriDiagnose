# AgriDiagnose local prototype

A working local web application with real pretrained crop-disease inference on CPU. No API key or cloud inference needed. Install dependencies and download the model once using the setup instructions below; subsequent inference works offline. Built from the supplied design materials, with the original HTML files preserved in `reference/`.

## Start here

Download **[AgriDiagnose-Project.zip](AgriDiagnose-Project.zip)** from this repository, extract it, and follow **Reproduce on another computer** below before launching the app. GitHub hosts the source and documents; this repository link is not a hosted diagnosis service. Earlier temporary presentation links have expired.

- [How the prototype works (Word)](AgriDiagnose%20How%20the%20Prototype%20Works.docx)
- [Prototype update presentation (PowerPoint)](AgriDiagnose%20Prototype%20Update.pptx)

The documents describe the prototype at the time they were written. Any dated temporary demo links in them are historical. Use the setup instructions here to run the current source.

## Run on this computer

After completing setup, double-click **Start AgriDiagnose.cmd** in this folder. The app opens at **http://127.0.0.1:8765**. Keep the server window open; press Ctrl+C there to stop. If Windows restricts PowerShell scripts, run the configured environment's Python with `app/server.py` instead of changing system execution policy.

1. Pick one of your seven example photos, a model-author reference photo, or a JPEG/PNG/WebP under 8 MB.
2. Select the actual crop and stage. Optional weather inputs are manual.
3. Select **Analyse leaf**. Review the top candidates and all gate results in **Inspect the pipeline**.
4. Optionally record a manual severity estimate. Download the JSON report or export a saved scan from browser history.

Photos are processed in memory by the local Python server and are not persisted. History stores results, filenames and context in browser local storage, limited to 30 scans. History does not store photos. Manual severity is captured by the current report export; the history entry is the original scan snapshot before subsequent edits.

## Implemented

- Responsive web UI, file chooser, drag and drop, supplied photos, separate reference examples.
- DINOv2-small quantized ONNX inference through ONNX Runtime CPU.
- 38-class model; advisory UI supports Tomato, Potato, Apple, Corn and Grape.
- EXIF rotation, RGB conversion, shortest side resized to 256, centre crop to 224, ImageNet normalization.
- Top-five scores from the full model output. Crop mismatches are rejected without renormalizing away competing classes.
- Blur rejection, training-feature similarity rejection and adjustable score gate.
- Rule-based follow-up context, optional manual severity, pipeline inspector, browser history and JSON exports.
- Loopback-only server, request-size limits, image validation, cross-origin browser POST rejection and restrictive content policy.

## What the prototype does not establish

The model is **pretrained by another author**, downloaded and run locally. We have not trained or validated an AgriDiagnose-specific model. It is a screening aid, not a reliable field diagnosis service. The model card describes controlled PlantVillage imagery and warns about field photos. Softmax scores are not calibrated probabilities of correctness.

The supplied seven photos all trigger at least one hold condition at the default gate. The blurry image is rejected for blur and distribution mismatch. Several other photos receive incorrect high-score predictions or disagree with the selected crop. Apple scab and grape black rot receive the expected named class but fail the distribution check. The model-author reference photos pass the checks. These observations demonstrate the benchmark-to-field problem in your deck; thresholds were not weakened to force examples to pass.

Severity is **manual**, not a learned regression output. There is no trained second head, segmentation model, location/weather integration, grounded LLM, or verified treatment database. Illustrative pesticide doses from the supplied prototype are not presented as actionable recommendations. Follow-up messages are prototype logic, not validated extension guidance.

## Reproduce on another computer

Use Python 3.12+:

```powershell
cd app
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe download_model.py
.\.venv\Scripts\python.exe server.py --open
```

The downloader uses pinned revisions and SHA-256 hashes from `app/models/manifest.json`. Inference executes ONNX graphs, not remote repository Python code. The current computer uses dependencies under `app/vendor/` and the bundled Codex Python. Model weights and installed dependencies are ignored by Git and remain on disk.

## Verification

```powershell
cd app
.\.venv\Scripts\python.exe verify.py
```

On this computer the bundled Python can also run `app/verify.py` directly. It runs all nine actual photos through the model, checks the reference examples, blur rejection, crop mismatch and a solid-colour negative case, and saves `app/evaluation.json`. This is a smoke test, not an accuracy estimate. No sample is classified from its filename.

## Research path

Collect independently labelled images with leaf/plant identifiers. Separate training, validation and field test sets by plant or leaf before augmentation. Fine-tune the candidate backbones and compare macro-F1 and field performance. Expert severity annotations are required to train the second regression head and report RMSE. Validate rejection thresholds on held-out images. Build region-specific extension rules before treatment advice. Keep these seven examples outside training if they remain evaluation examples.

## Model provenance and attribution

- Model: [rodynaemad/plant-disease-dinov2-small](https://huggingface.co/rodynaemad/plant-disease-dinov2-small), by Rodayna Emad Mohamed. Model revision `06ebf5e308e100fd000d738c42b059ebb5f58a16`.
- Matching centroids, rejection threshold and two reference images: [model author's scanner](https://huggingface.co/spaces/rodynaemad/plant-disease-scanner), revision `b26e704bd72c2fe05632ea6f7be680a2eca0d8f5`.
- Model card declares CC BY-SA 4.0, based on PlantVillage (Hughes & Salathé, 2015; Mohanty, Hughes & Salathé, 2016). DINOv2 backbone: Apache 2.0. Preserve these attributions and comply with the source licenses when redistributing. The original model card is retained at `app/models/MODEL_CARD.md`.
- User-supplied photographs are included unchanged as prototype test fixtures; their original reuse rights have not been established. No ownership or new license is claimed for these images. The application runs locally, matching the requested local model workflow.

## Repository package

The complete source tree is inside `AgriDiagnose-Project.zip`, including the frontend, Python backend, sample images, tests, model download manifest, original HTML references and documentation. Model weights and installed dependencies are excluded; the setup script downloads the pinned model.
