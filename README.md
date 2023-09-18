# EMtools
![Visualization of raw, predicted and final segmentation](https://i.imgur.com/hX9HMJN.jpeg)

## Intro
This collection of scripts is meant to be used to preprocess, segment, and analyze 2D TEM images of myelinated fibers. It is very much a work in progress and bugs are expected. Nevertheless, the pipeline usually runs well and delivers (imo) satisfactory results. The scripts in this repository are meant to be used in conjunction with the Uni-EM installation on the Höllenmaschine 2.0 (Room 2101) of the Paul Flechsig Institute in Leipzig. 

The whole analysis pipeline consists of four core steps, including an optional validation step:
  1) Preprocessing. This step applies a contrast-enhancing filter to your raw images, enhancing the local contrast while keeping details.
  2) Prediction by the neural net. This neural net is trained to classify your images in three categories: Myelin, Fiber, and Background. The output is of the same shape as your input data, with each pixel intensity value corresponding to one of these three classes (Semantic segmentation).
  3) Postprocessing. In this step, the Semantic segmentation (which pixel is of myelin/fiber/background class?) is turned into an instance segmentation (which of these pixels are part of a cell?). This means that through a series of image analysis steps, neighbouring instances of the same class (e.g. two myelin sheaths touching) are detached from each other and individually labeled. After that, a series of measurements are taken from each cell and results are saved as images and a .csv file.
  4) Optional: Validation. This step requires a manually labeled "ground truth" dataset of your specific data and outputs various validation measures, which your reviewers will probably want to see.

Disclaimer: The model was trained on manually labeled data from human corpus callosum samples. Therefore, this is where it should work best. Our testing in human superficial white matter and rat and hamster cortex and spinal cord showed that it also yields satisfactory results when used somewhere else, as long as the tissue contains myelinated fibers (the more, the better). A good rule of thumb is: The more similar your data is to the image shown above, the better the whole pipeline will perform.


---


## Step-by-Step
The whole process takes maybe 30 minutes for a few small images (<100MB), longer for more. For most use cases, all of these data analysis steps will fit within half a day or so.

### 0) Prepare your 2D TEM images 
   - If you haven't, download this repository and place it into your personal folder on on Hoellenmaschine 2.0.
   - Your raw TEM images should be (approximately) 3000x magnification, in .tif or .png format. Put them all into the provided `0_raw` folder in the repository.


### 1) Run the preprocessing
   The preprocessing script (`1_preprocessing.ipynb`) applies contrast adapted histogram equalization (CLAHE, see e.g. https://imagej.net/plugins/clahe) and then resaves your data to a different directory.
 - Navigate to your downloaded `EMtools-main` folder
 - Hold the `shift key`, right click in an empty space inside the folder and click `open powershell window here`
 - Type `jupyter notebook`. This should open a browser window with a file browser. Continue in this file browser.
 - Open `preprocessing.ipynb`
 - If your images are small and/or lower magnification than 3000x, you may want to set the downscale factor `ds`to 1 or 2 instead of 4.
 - Make sure you placed your raw images into the `0_raw` folder. Then, press the double arrow at the top of the window and then press `Restart and run all cells`.
 - After the script is finished, your preprocessed and re-saved data should be in the `1_preprocessed` folder.
 - Keep the browser open for later.

Note: Keep in mind that your images will be cropped to 1024x1024 or 2048x2048 pixels (after downscaling) depending on their size, since these are the only dimensions the neural net accepts.

### 2) Predict the data using Uni-EM
- Open Uni-EM on the desktop (`Uni-EM\main.exe`) - starting this might take a minute
- Open the following three folders by dragging and dropping them into the Uni-EM window. There is no feedback to this from the GUI window, only in the accompanying terminal:
    1) The folder your preprocessed images are in: `1_preprocessed`
    2) An empty folder you want your predicted images to be in: `2_predicted`
    3) The model folder `E:\AG_Morawski\Philip\EM\20230419_densenet_12_12_20` (Please *do not* move this folder)
- Click Segmentation -> 2D DNN and then click on the inference tab
- Select the three folders you already opened above as Image folder, Model Folder, and Output Segmentation folder (only the correct folder will appear in each of the dropdown menus).
- Select the maximum maximal unit size (2048)
- Click "Execute" and watch the model do its work in the terminal (or don't and grab a coffee)
- Once the script is done, it will display `Inference finished` in the terminal. 

### 3) Run the postprocessing
- Return to the Jupyter window in the browser, which you opened in step 1. Open `3_postprocessing.ipynb`.
- Specify the pixel size in the variable `px_size` if you want your outputs to be correctly scaled.
  - Side note: The output from the DNN is a classification of your data into three categories: 1) Background 2) Fiber and 3) Myelin. All three of these classes are coded as intensities in the output image. Depending on how many "uncertain" pixels you want to include in downstream analysis, you can try varying the threshold values that.
- Click the double arrows at the top of the jupyter notebook to run it
- The script will save save processed versions of each image in `3_postprocessed`. You will also get a .csv and .xlsx file containing all of your measures once everything is done, both of which include all of your measured results, and where you can find each given cell (file name and slice)
- For a customized analysis in other programs (FIJI, your own python script, etc..), the raw binary masks are also included in the results (*filename*_outer_labeled.tif and *filename*_inner_labeled.tif).


### 4 (optional) Run the validation
- To get measures of how good the model performs on your data, you will need to validate the model. For this purpose, you can use `postprocessing_val.ipynb`. To do so, you will need a manually labeled version of your data and the models raw prediction results (which are generated after step #2).
- manually labeled data can be generated e.g. with GIMP (https://www.gimp.org/) or a variety of other image manipulation tools. It should be analogous to the included validation.png.
- To run the validation script, replace the present files with your own data and press the double arrow at the top of the ipynb.
- After running the script, you will get a bunch of different measures telling how well the whole pipeline works on your data.
