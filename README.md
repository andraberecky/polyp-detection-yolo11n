# Polyp Detection with YOLO11n (Kvasir-SEG)

In this project I trained a YOLO11n model to detect polyps in endoscopy images. The Kvasir-SEG dataset comes with segmentation masks, not bounding boxes, so part of the work was converting the masks into the YOLO format myself.

This is a personal project for learning object detection. It is not meant for any medical use.

## Results

I evaluated the best checkpoint on a test set of 150 images that the model never saw during training.

| Split      | Precision | Recall | mAP50 | mAP50-95 |
|------------|-----------|--------|-------|----------|
| Validation | 0.904     | 0.773  | 0.814 | 0.646    |
| Test       | 0.907     | 0.798  | 0.848 | 0.700    |

On a Tesla T4 (Google Colab), inference takes about 5 ms per image at 640 px. The model is small, around 2.6M parameters.

## What I did

1. Checked that every image in the dataset has a matching mask (1000 pairs).
2. Converted the masks to bounding boxes. I thresholded each mask, found the contours with OpenCV and took the bounding rectangle of each one. The boxes are saved in YOLO format with a single class, `polyp`.
3. Split the data randomly into 70% train, 15% validation and 15% test, with a fixed seed so it can be reproduced.
4. Trained YOLO11n starting from the pretrained weights: 100 epochs, early stopping with patience 20, image size 640, batch size 16.
5. Evaluated on the validation and test sets and compared a prediction with the ground-truth mask for a test image.

## How to run it

The notebook was written and run in Google Colab with a T4 GPU.

1. Open `notebooks/kvasir_yolo11n.ipynb` in Colab (or locally if you have a GPU).
2. Install the requirements with `pip install -r requirements.txt`.
3. Run all the cells. The notebook downloads the dataset from the [Kvasir-SEG page](https://datasets.simula.no/kvasir-seg/), builds the YOLO folders and trains the model.

I did not upload the dataset or the training runs to this repository because they are large.

## Things I would be careful about

- The train/val/test split is random over images. The dataset has no patient IDs, so similar images from the same patient could end up in both train and test. This means the results might be a bit better than they would be on completely new patients.
- The boxes come from the masks, they were not drawn by hand. If a mask has several separate regions, I get several boxes.
- Recall is lower than precision, so the model misses some polyps. I used the smallest YOLO11 version and the dataset is small, so a bigger model or more augmentation might help. I did not test that yet.
- All numbers come from one training run with one seed.

## Dataset and credits

Dataset: Kvasir-SEG

Jha, D., Smedsrud, P.H., Riegler, M.A., Halvorsen, P., de Lange, T., Johansen, D., Johansen, H.D. *Kvasir-SEG: A Segmented Polyp Dataset*. MultiMedia Modeling (MMM 2020), LNCS 11962, pp. 451-462, Springer, 2020. https://doi.org/10.1007/978-3-030-37734-2_37

Model: [Ultralytics YOLO11](https://github.com/ultralytics/ultralytics) (AGPL-3.0)
