See prior talk, [[Training Object Detection in Create ML - 19]]

#createml 

# Improvements
* Smaller model size
* Less training data
* More configuration options

# Demo
Object Detection template in CreateML.

Data/config options can be tweaked here.  

## Training data
Must be stored in a folder which contains all training images and the annotations.json file.  

Each object annotation consists of the object's label and its location in the image.  The location is based on the topleft origin.

## New training parameters.
### Algorithm

* Full network.  Default training algorithm since the beginning.  Based on YOLOv2.  All parameters are trained using your data.  The resulting coreml model encodes all the learned parameters.  Model size can be halved due to quantization.  This algorithm is recommended when you have large amounts of data, like >200 samples per class.  Provides backwards compatible to iOS 12.  
* Transfer learning.  Leverages ML models already in the OS.  ex model, "object print".  Only the "head network" (part after 'object print' or similar model) is trained on your data, so we only need to store parameters for those.  coreml model contains only "head network" parameters which is 5x smaller.  Great option for limited data, smaller model size.  80 examples/class.  iOS14+


### iterations
Number of times parameters are updated.  A default value is picked based on dataset size.  You can increase iterations if the model has not converged or reduce if model is doing well early.

### batch size
Number of training examples per iteration.  Default value chosen based on hardware restrictions.  Higher batch size is better.

Use the default value or reduce based on perf restrictions

### grid size
Model leverages a grid.  Defines the aspect ratio of the images and where the model will look.  Image will be resized to fit the grid (e.g. without preserving aspect ratio).

Network produces predictions for each grid cell.  Each prediction contains

* object detected y/n
* predicated class
* coordinates

yolo works fine for multiple objects, but each object is associated with 1 cell.  Since each cell can only pick 1 class, it is forced to pick if both is in a cell.

to predict both, anchor boxes are defined.  Have a set aspect ratio and detect multiple objects within a grid cell.  

createml has a default dimension of 13x13, totaling 169 cells.  15 anchor boxes of varying ratios are evaluated for each cell.  Therefore the default model is doing a total of 2535 predictions per image.

Consider the case of fixed object size like dice.  If your grid size is larger than the dice, multiple dice may appear in a grid cell.  Since each grid cell can only detect one object, not all dice will be detected.

For a non-square input image, using a square image can distort th natural shape of objects.  May prevent the model from capturing fine-grained patterns.  Choosing a grid size similar to aspect ratio may be superior.

# Demo
Snapshots are helpful in checking on training progress.  

# Evaluation tab
We want correct labels, also the right locations.
Getting the bounding boxes to exactly match is hard.  A number that captures how close they are is necessary.

Intersection / Union (I/U).  0% = no overlap, 100% = perfect overlap.  

For a prediction to be considered correct it needs to have class label, and IU score above some threshold.  If either is less, the prediction is incorrect.  This information is used to compute a metric called mean average precision (MAP).

# Demo

# Wrap up
* easy to use
* customizable
* Reduces data requirements
* Smaller coreml model

