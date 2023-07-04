# EMtools
![Screenshot 2023-07-04 150311](https://github.com/PhilipRuthig/EMtools/assets/39408485/22e879d4-824f-43ff-afe1-680d67efe555)

## Intro
This collection of scripts is meant to be used to preprocess, segment, and analyze 2D TEM images of myelinated fibers. It is very much a work in progress and bugs are expected. The scripts in this repository are meant to be used in conjunction with the Uni-EM installation on the Höllenmaschine 2.0.

Note: The model was trained on manually labeled data from human corpus callosum samples. Therefore, this is where it works best. Some testing in human superficial white matter and rat and hamster cortex showed that it also yields okay results when used somewhere else, as long as the tissue contains myelinated fibers. The more similar your data is to the image shown above, the better.

## Step-by-Step
### 1) Prepare your 2D TEM images 
   Your TEM images should be in .tif or .png format. Put them all into a single folder without sub-folders.

### 2) Run the preprocessing
   The preprocessing script applies contrast adapted histogram equalization (CLAHE, see e.g. https://imagej.net/plugins/clahe) and then resaves your data. To run it, follow these steps:
 - Press windows + R
 - Type in `powershell`
 - Navigate to your (personal) folder where you cloned/downloaded this repository to
 - Type `jupyter notebook`. This should open a browser window with a file browser. Continue in this file browser.
 - Open `pre_uni-em.ipynb` 
 - For `path_input`, put the path where you saved your raw images
 - For `path_output`, put the path where you would like your preprocessed images to be in.
 - Click the double arrow at the top of the jupyter notebook to run it.
 - Keep the jupyter server in the browser open for later.

### 3) Predict the data using Uni-EM
- Open Uni-EM on the desktop (`Uni-EM\main.exe`) - starting this might take a minute
- Open the following three folders by dragging and dropping them into the Uni-EM window. There is no feedback to this from the program, only in the accompanying terminal.
    1) The folder your preprocessed images are in: `path_results` as described above
    2) An empty folder you want your predicted images to be in
    3) The model folder `E:\AG_Morawski\Philip\EM\20230419_densenet_12_12_20`
- Click Segmentation -> 2D DNN and then highlight the inference tab
- Select the three folders you already opened above as Image folder, Model Folder, and Output Segmentation folder.
- Select the maximum maximal unit size (2048)
- Click "Execute". The whole process should take a few minutes to an hour, depending on how many images are in the folder.
- Once the script is done, it will display `Inference finished` in the Uni-EM terminal.

The output files classify your data into three categories: 1) Background 2) Fiber and 3) Myelin. All of these three classes are coded as intensities in your greyscale output image. Depending on how many of the cases where the model was uncertain you want to include, you can vary the initial threshold you want to include in further analyses. I suggest using the following values: Myelin > 55, Fibers >24 and <40, Background is <=24. 

### 4) Run the postprocessing script.
- Return to the Jupyter Server window in the browser, which you opened in step 2. Open `post_uni-em.ipynb`.
- Insert the path where your predicted images were saved to the variable `seg_path`.
- Create an empty folder and insert its directory to `path_results`. This is where your results and plots will be created.
- Click the double arrows at the top of the jupyter notebook to run it
- The script iterates through all of your data and saves processed versions of each image in `path_results`. You will also get a .csv file once everything is done, which includes all of your results.

### 5) (optional) Run the validation
- For the validation, you will need a manually labeled dataset, your raw prediction results and `post_uni-em_val.ipynb`. Reach out to Philip for details. Documentation will come in the future.
