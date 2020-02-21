# Multi-Gesture Recognition
A set of scripts to train a convolutional neural network from scratch to recognize multiple gestures in a wide range of conditions with TensorFlow and a Raspberry Pi 3/4.

## Getting started

This is the example code used in Arm's [Raspberry Pi multi-gesture recognition walkthrough](https://developer.arm.com/technologies/machine-learning-on-arm/developer-material/how-to-guides/teach-your-pi-multi-gesture) - full installation and usage instructions can be found there.

*Note: These example scripts are designed to be easy to read and follow, not to demonstrate state-of-the-art or best practice.*

## Dependencies

From a base Raspian install you will need to add TensorFlow dependencies:

    sudo apt-get install libblas-dev liblapack-dev python-dev libatlas-base-dev gfortran python3-setuptools python3-h5py 
    
    # Replace this URL with the current version:
    sudo pip3 install tensorflow

## Example

Check out the [multi-gesture recognition walkthrough](https://developer.arm.com/technologies/machine-learning-on-arm/developer-material/how-to-guides/teach-your-pi-multi-gesture). If you just need to be reminded about the syntax supported by the example scripts, read on.

### Preview 
Make sure the camera can see the area of interest (the rotation doesn't really matter):

    python3 preview.py

### Record

Record a single walkthrough of all the different actions the system should learn to recognise, stopping after a given number of seconds (-1 to continue until ctrl-c is pressed):
    
    python3 record.py day1 -1
    
This stores the video as PNG files in the day1/ directory. If ctrl-c is pressed the last image file may be corrupt.

If you are recording a test dataset (see below), consider modifying the training_mode=True parameter passed to Camera to False before running. This disables to random white balance fluctuations and will give you images more representative of the real-world but less useful for training on.
    
### Classify

Classify these using an interactive GUI:

    python3 classify.py day1

Keyboard controls are:

* 0-4: classify the current image as '0' through '4'
* 9: classify the next 10 images as '0' (useful if 0 is the 'normal' class of which there are often many frames)
* Escape: go back one image
* Space: toggle viewer brightness to make human classification easier (does not modify the image file)
* S: save changes by moving images to their new directories and quit
* Close the window without pressing S to exit without moving files.

Files from day1/ are classified and moved into day1/0/, day1/1/, day1/2/ ... subdirectories.

### Merge

If you have recorded and classified multiple 'days', merge them into a single dataset for training or testing:
    
    python3 merge.py day1+2 day1 day2
    
This copies all files in day1 and day2 into day1+2 whilst preserving directory structure and introducing unique names to avoid overwriting/conflicts.

### Create Validation Set

Split a classified dataset into two - keep 90% of the files the original set for training and create a new dataset prefixed by val_ with the other 10%:
    
    python3 validate_split.py day1
    
### Train

Train a model from scratch using a training and validation dataset:

	python3 train.py day1 val_day1
	
The best model found during training (as measured by its performance on the validation dataset) is saved as day1/model.h5.

It's recommended to copy the dataset to a more powerful machine (e.g. laptop or desktop) and run train.py there. Training a convolutional neural network from scratch can easily take an hour on such a machine and would need much longer on a Raspberry Pi.

### Test

Test the model trained on one dataset on the images in another dataset and report its accuracy and the images that it mispredicted:

    python3 test.py day1 day2
    
The above assumes day1/model.h5 has been created with train.py. It is good practice to record one or more tests set (e.g. test1, test2) that are never trained on and are used to evaluate how well a model is learning to generalize to new data.

### Run a model

Run a trained model on the Raspberry Pi and print recognised gestures and their probabilities:

    python3 run.py model.h5

### Perform actions based on gestures

The script used to drive the video demo is included, but cannot be used without modifiation:

    vim story.py
    python3 story.py model.h5
    
The Room class contains the code that must be modified to perform your own actions. For example, audio/epic.mp3 is not included in the distribution and neither is a machine called "zero" with ./lock and ./unlock scripts!
