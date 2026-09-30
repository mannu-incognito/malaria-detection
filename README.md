# Malaria Detection Using Image Processing Techniques

Classifies a blood smear cell image as **Parasitized** or **Uninfected** using transfer learning (EfficientNetB0) on the [Cell Images for Detecting Malaria](https://www.kaggle.com/datasets/iarunava/cell-images-for-detecting-malaria) dataset.

Everything runs on **Google Colab**, so you don't need to install anything or own a GPU. The model is trained once, saved to Google Drive, and reloaded on later runs.

[![Open Training Notebook In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/YOUR_REPO/blob/main/malaria_detection_colab.ipynb)
[![Open Prediction Notebook In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/YOUR_REPO/blob/main/malaria_predict_colab.ipynb)

> Replace `YOUR_USERNAME` and `YOUR_REPO` in the two badge links above with your own GitHub username and repository name.

---

## Project structure

```
.
├── malaria_detection_colab.ipynb   # train, evaluate and save the model
├── malaria_predict_colab.ipynb     # load the saved model and test your own images
├── Malaria_Detection_Intermediate_Report.docx
└── README.md
```

## How it works

| Step | What happens |
|------|--------------|
| Data | Downloads the dataset (27,558 cell images, two balanced classes) and removes duplicate files |
| Split | Stratified 70% train / 15% validation / 15% test, fixed seed |
| Preprocessing | Resize to 128x128, RGB, random flips / rotation / zoom / contrast / brightness (training only) |
| Model | EfficientNetB0 (ImageNet weights) + global pooling + dropout + sigmoid output |
| Training | Stage 1: train the head. Stage 2: fine-tune the whole network at a low learning rate |
| Evaluation | Precision, recall, F1, accuracy, ROC-AUC, confusion matrix on the held-out test set |
| Saving | Model stored in Google Drive, so no retraining is needed next time |

## Setup and run on Google Colab

### Part 1: Train the model (one time only)

1. Push this repo to GitHub, or download the two `.ipynb` files.
2. Open `malaria_detection_colab.ipynb` in Colab, either with the badge above, or at [colab.research.google.com](https://colab.research.google.com) via **File → Upload notebook** (or **GitHub** tab, then paste your repo URL).
3. Turn on the GPU: **Runtime → Change runtime type → T4 GPU → Save**.
4. Run everything: **Runtime → Run all**.
5. When asked, allow access to your Google Drive. This is where the model gets saved.
6. Wait for training to finish (roughly 10 to 20 minutes on a T4). The notebook then prints the test metrics and draws the confusion matrix, ROC curve and sample predictions.

After the run you'll have this folder in your Google Drive:

```
MyDrive/malaria_project/
├── malaria_model.keras   # final trained model
├── malaria_best.keras    # best checkpoint from fine-tuning
├── stage1.keras          # checkpoint from head training
├── history.json          # accuracy history for the training curve
└── results.png           # confusion matrix, ROC curve, accuracy plot
```

**Running it again:** the notebook detects `malaria_model.keras` and loads it instead of training. To retrain from scratch, delete that file from the Drive folder.

### Part 2: Test your own images

Use `malaria_predict_colab.ipynb`. It has no dataset download and no training, and works on a normal CPU runtime.

1. Open the notebook in Colab and run the cells from the top.
2. **Load the model:** it is read from `MyDrive/malaria_project/malaria_model.keras`. If you're on a different account, skip the Drive cell and upload `malaria_model.keras` from your computer when asked.
3. Run the **Test images** cell, click **Choose files**, and select one or more cell images. You get the label and a confidence score for each.
4. *(Optional)* Run the **Web app** cells to get a public link with drag-and-drop upload. Handy for a live demo.

**Sharing with someone else** (for example your professor): share the `malaria_project` Drive folder with them, or send them `malaria_model.keras` plus `malaria_predict_colab.ipynb`.

## Kaggle download

The dataset is fetched automatically with `kagglehub`. If Colab asks for Kaggle credentials:

1. Create an API token at kaggle.com → Settings → **Create New Token**.
2. In Colab, open the key icon (Secrets) on the left and add `KAGGLE_USERNAME` and `KAGGLE_KEY` with the values from the token. Turn on notebook access for both.
3. Re-run the download cell.

## Results

Fill this in from the evaluation cell after your training run.

| Metric | Value |
|--------|-------|
| Accuracy | ____ |
| Precision (Parasitized) | ____ |
| Recall (Parasitized) | ____ |
| F1-score | ____ |
| ROC-AUC | ____ |

## Tips and troubleshooting

- **`Found saved model` but I want to retrain:** delete `malaria_model.keras` from `MyDrive/malaria_project`.
- **Training is very slow:** you're probably on a CPU runtime. Switch to a T4 GPU and run again.
- **Colab disconnected mid-training:** the best checkpoint is still in Drive. Reconnect and run all again. The notebook will retrain from the beginning, but you can load `malaria_best.keras` manually if you want to keep that progress.
- **Out of memory:** lower `BATCH` from 64 to 32 in the config cell.
- **Predictions look wrong on my image:** the model was trained on single-cell crops like the ones in the dataset. Photos of a whole microscope slide will not give meaningful results.
- **Keras version errors when loading the model:** use the same Colab environment for both notebooks, or retrain in the current one.

## Limitations

- The dataset has no patient IDs, so cells from the same patient can land in both training and test sets. Performance on new patients may be a bit lower than the test score.
- Works on pre-segmented single cells only. It does not locate cells in a full slide.
- Trained on P. falciparum slides from one source, so other stains, cameras or species may reduce accuracy.
- This is an academic project and **not a medical device**. Do not use it for clinical decisions.

## Future work

- Grad-CAM heatmaps to show where the model is looking
- Comparison with a classical pipeline (thresholding + texture features + SVM) and other backbones
- Patient-level validation and threshold tuning for higher recall
- Whole-slide detection with an object detector, and a TensorFlow Lite version for mobile

## References

- Rajaraman, S. et al. (2018). *Pre-trained convolutional neural networks as feature extractors toward improved malaria parasite detection in thin blood smear images.* PeerJ, 6, e4568.
- Tan, M. and Le, Q. V. (2019). *EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks.* ICML.
- Dataset: [Cell Images for Detecting Malaria (Kaggle)](https://www.kaggle.com/datasets/iarunava/cell-images-for-detecting-malaria)
