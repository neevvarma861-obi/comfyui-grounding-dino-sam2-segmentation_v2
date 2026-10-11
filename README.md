# Text-Prompted Object Segmentation with SAM 2 and GroundingDINO (ComfyUI)

## Project Overview

This project segments a chosen object from an image using a text prompt. GroundingDINO finds the object from the text description, and SAM 2 (Segment Anything Model 2) turns that detection into a precise pixel mask. The workflow produces two outputs per image: a black-and-white **mask** and a **cutout** of the object.

The workflow was tested on three images:

| # | Image | Text prompt |
|---|---|---|
| 1 | Tree in a moorland landscape (`29339.jpg`) | `tree` |
| 2 | Headphones | `headphones` |
| 3 | Flower | `flower` |

## Repository Structure

```
.
├── README.md
├── workflow/
│   └── assessment3_sam2_workflow.json
├── input/
│   ├── 29339.jpg
│   ├── <headphones image>
│   └── <flower image>
├── output/
│   ├── tree_seg_mask.png
│   ├── tree_seg_cutout.png
│   ├── headphones_seg_mask.png
│   ├── headphones_seg_cutout.png
│   ├── flower_seg_mask.png
│   └── flower_seg_cutout.png
└── screenshots/
    └── workflow_graph.png
```

## Dataset Structure

This is an inference-only task, so there is no training dataset.

| Item | Details |
|---|---|
| Input | Three images (tree, headphones, flower), each with one text prompt |
| Output | For each image: a binary mask (`seg_mask`) and a cutout (`seg_cutout`) |

## Models Used

| Component | Model |
|---|---|
| Segmentation | SAM 2: `sam2_hiera_large.pt` |
| Text-to-box detection | GroundingDINO: `GroundingDINO_SwinT_OGC` (694 MB) |

## Installation Steps

1. Install [ComfyUI](https://github.com/comfyanonymous/ComfyUI) and run it once to confirm it starts.
2. Install the custom node pack **comfyui_segment_anything** (the nodes marked "segment anything2") from the [storyicon repository](https://github.com/storyicon/comfyui_segment_anything), either through **ComfyUI Manager** (search "segment anything") or by cloning it into `ComfyUI/custom_nodes/` and running `pip install -r requirements.txt`.
3. Restart ComfyUI.
4. The SAM 2 and GroundingDINO models are downloaded automatically the first time you select them in the loader nodes (internet connection needed). They are stored in `ComfyUI/models/sam2/` and `ComfyUI/models/grounding-dino/`.
5. Copy the input images into `ComfyUI/input/`.

## Training Instructions

Not applicable. No model was trained or fine-tuned. Both models are used as released.

## Prediction Instructions

1. Open ComfyUI in the browser.
2. Drag `workflow/assessment3_sam2_workflow.json` onto the canvas.
3. In **Load Image**, choose the input image.
4. In **GroundingDinoSAM2Segment**, set the `prompt` for that image:
   - tree image: `tree`
   - headphones image: `headphones`
   - flower image: `flower`
5. Click **Run**. Outputs are saved to `ComfyUI/output/` as `seg_mask_*.png` and `seg_cutout_*.png`.
6. Repeat steps 3 to 5 for each image.

## Workflow Description

1. **Load Image** reads the input.
2. **SAM2ModelLoader** and **GroundingDinoModelLoader** load the two models.
3. **GroundingDinoSAM2Segment** takes the image and the text prompt. GroundingDINO detects the object (confidence `threshold` 0.3), and SAM 2 produces the mask.
4. The node returns the **cutout image** (saved as `seg_cutout`) and the **mask**.
5. **MaskToImage** converts the mask to a visible black-and-white image, which is saved as `seg_mask` and also shown in a preview.

## Image Preprocessing

No manual preprocessing is applied. Images are passed to the node at their original size.

## Parameters

| Parameter | Value | Effect |
|---|---|---|
| `prompt` | per image (see table above) | Text description of the object to segment |
| `threshold` | 0.3 | Detection confidence. Lower detects more objects (and more false positives), higher detects fewer |

## Evaluation Results

Evaluation is visual, as no ground-truth masks were provided.

| Image | Object found | Mask quality | Notes |
|---|---|---|---|
| Tree | Yes | Good |
| Headphones | Yes | Good | 
| Flower | Yes | Good | 

Sample outputs are in the `output/` folder.

## Known Limitations

- Results depend on the text prompt. Vague prompts can pick the wrong object, and specific prompts (for example `windswept tree`) often work better.
- If several objects match the prompt, they can be merged into one mask.
- A fixed threshold of 0.3 may not suit every image. Raise it if there are false detections, lower it if the object is missed.
- Thin or fine structures (leaves, grass, headphone headband edges) can have rough or incomplete mask edges.
- The cutout is a hard-edged mask with no feathering, so edges may look slightly jagged when placed on a new background.
- The first run needs an internet connection to download the models.

