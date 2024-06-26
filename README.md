# Wearable Cognitive Assistance Student Project Guide
In this class, you will build your own wearable cognitive assistance (WCA) "Sandwich" application. This guide is intended to walk you through the application creation and testing process. At the end of the exercise, you should have a working "toy" application that includes all the current capabilities of CMU's WCA application platform.

Each student will be assigned to a team. The **team specific information is provided at the end of this guide**. Each team will execute the following steps in creating the application:

1. **Data Collection** -- Use your smart phone to collect videos of the sandwich components and sub-assemblies. These videos will be used in the following two steps to create a training and testing dataset for a WCA object detector.
2. **Video Upload** -- Upload your videos to the CVAT annotation tool.
3. **Annotate the Dataset** -- Use CVAT to convert the videos into a labeled dataset.
4. **Prepare the Dataset for Training** -- Download the dataset from CVAT onto the cloudlet that will be used for training an object detector.  Use the *OpenTPOD Tools* and *datumaro* to clean and filter the dataset.
6. **Train the Object Detector** -- Use *Yolo* to train an object detection neural network model for your application.
7. **Define the Application Logic** -- Use the *OpenWorkFlow* tool to define a finite state machine (FSM) and corresponding user prompts for your application.
8. **Install the FSM and object detector in your WCA Backend** -- Move the application-specifc `*.pbfsm` and `*.pt` files into the right locations on the cloudlet.
9. **Run the WCA Backend** -- Start the *GatingWCA* server on the cloudlet. This server executes your application logic.
10. **Connect your WCA Client to the Backend** -- Launch the client application which will establish a connection to your backend GatingWCA server.

![image](https://github.com/cmusatyalab/wcacourse/assets/6760112/cba8bd86-5c5a-4b8b-8c6f-00f5f104f6c8)

At this point, you are ready and able to actually use your sandwich application. You may find that you need to iterate some or all of these steps in order to get an acceptable detector and tune your FSM.

These steps are described in more detail below. 

Your development environment consists of four components:
1. A *linux laptop* that you will use to connect to CVAT, OpenTPOD, OpenWorkFlow, and the GatingWCA server.
2. A *cloudlet (actually, an AWS VM)* that is preconfigured with the credentials and components you need.
3. An *Android smartphone* that serves as a video data capture device and as the WCA application client.
4. A *sandwich kit* that your application will assist a naive user to create.

## Data Collection
Take video(s) of a total of 1-2 minutes in length using your smartphone camera. Videos should contain the first object individually (*usually, the bread*) and the sub-assemblies of objects.

### *Pro Tips*
* Take your videos from a variety of directions and orientations.
* When creating the videos, use the same background and lighting that you will be use when running the application.

## Video Upload
Using the browser on the smartphone:
* Navigate to [CVAT](https://cvat.cmusatyalab.org/auth/login) and login.
* Create a new project and open it. Make a note of the project number. See image below.

![image](https://github.com/cmusatyalab/wcacourse/assets/6760112/dc5138ed-50f2-45a5-8611-645cf6e8f154)

* Create a new task. Before opening it, upload the videos you captured into the task.

## Annotate the Dataset
In this step, you are labeling the objects and sub-assemblies in your application. You will use [CVAT](https://docs.cvat.ai/docs/) to do the labeling (*aka annotation*).
* Using the browser on the laptop, navigate to [CVAT](https://cvat.cmusatyalab.org/auth/login) and [login](https://github.com/cmusatyalab/wcacourse/edit/main/README.md#team-specific-information).
* Open the project you created from your phone.
* Add the labels that you will use in your project. Now, open the task and the job shown.
* Add labels to your video starting from the beginning.
* When you are done labeling, `Save`. Then, go to the `Jobs` page, find your job, click the three dots and select `Finish Job`.

For more information on how to use CVAT, see the [CVAT](https://docs.cvat.ai/docs/) documentation.

### *Pro Tips*
* You will need a task for each video that you label.
* You can select the method (e.g., rectangle, polygon) you will use to draw bounding boxes around your objects when you create the label. If you select `Any`, you can use different methods as you annotate. Polygons are more accurate than other methods. Rectangles are easier to reposition.
* The better your bounding boxes, the more accurate your detector will be.
* Make sure you use `Track` when you initiate labeling of a particular object.

## Prepare the Dataset for Training
SSH into your cloudlet from the laptop and then download and manipulate the dataset using opentpod-tools and datumaro. See the [opentpod-tools](https://github.com/cmusatyalab/opentpod-tools) for more info.

```sh
# On laptop
> ssh -i ~/.ssh/wca-student.pem wcastudent@<CLOUDLET_NAME>
# On cloudlet
$ source venv-opentpod/bin/activate
# Download the dataset
$ tpod-download --project <PROJECTNO> --url https://cvat.cmusatyalab.org --username <CVATUSERNAME> --password <CVATPASSWORD>
```
Find the directory called `datumaro_project_<YOURPROJECTNO>`. This is your `DATASETNAME`. Now, remove frames with no annotations:
```sh
$ tpod-filter <DATASETNAME>
```
You should see a new dataset directory called `filtered`. Remove frames with duplicate information:
```sh
$ tpod-unique filtered
```
You should see a new dataset directory called `unique`. Split the dataset into training and validation sets with 90% of the samples used for training and 10% for validation.
```sh
$ datum transform -t random_split -o split unique -- -s train:0.9 -s val:0.1
```
You should see a new dataset directory called `split`. You can inspect this dataset with:
```sh
$ datum dinfo split
```
Convert the dataset into the format used for the `yolo` object detector.
```sh
$ datum convert -i split -f yolo_ultralytics -o yolo-dataset -- --save-media
```
The `yolo-dataset` is now ready to be used in training. If you iterate back through this section, you will need to remove all the directories created before you begin.

## Train the Object Detector

Train a yolo model using the curated dataset. For more information on yolo, see [here](https://docs.ultralytics.com/models/yolov8/). The following command trains a yolov8 object detector on your dataset over 100 epochs. 

```sh
$ yolo detect train data=$(pwd)/yolo-dataset/data.yaml model=yolov8n.pt epochs=100 imgsz=640 project=yolo-project
```
While training is happening, feel free to move to the *Define the Application Logic* step.

The results of the training can be found in `yolo-project/train`. The model you will use later is `yolo-project/train/weights/best.pt`. Copy this file to `/home/wcastudent/models`.
```sh
$ cp  yolo-project/train/weights/best.pt /home/wcastudent/models
```

### *Pro Tips*
* If you want to see the training results, inspect `yolo-project/train/results.csv`. The images in this directory show various plots of the training metrics. To view these plots, `scp` the files back to your laptop.
* After the first training iteration, subsequent training results will be in `yolo-project/train<ITERATION_NO>`. Make sure to update the model in `/home/wcastudent/models` after each successful iteration.


## Define the Application Logic
Use the *OpenWorkFlow* tool to define a finite state machine (FSM) and corresponding user prompts for your application. 

[OpenWorkFlow Documentation](https://github.com/cmusatyalab/OpenWorkflow) 

[Access OpenWorkFlow](https://cmusatyalab.github.io/OpenWorkflow/)

From OpenWorkFlow, create your application finite state machine:
1. Create a `Start` state.
2. Create your first task state (e.g., `Bread`). Add a `YoloProcessor` to this state. Make sure you add in the conf_threshold. Leaving the default will cause an error. The model_path should be `/home/wcastudent/models/best.pt`.
3. Add a transition between the Start state and the Bread state. Enter in an `Audio Instruction` for the task that the user should do to progress to the Bread state (e.g., *Put the Bread on the Table*). Add an "Always" predicate.
4. Create your second task state (e.g., `Bread-Lettuce`) with a `YoloProcessor`. Make sure you add in the conf_threshold.
5. Add a transition between the Bread state and the Bread-Lettuce state. Enter in an `Audio Instruction` for the task that the user should do to progress to the Bread-Lettuce state (e.g., *Put the Lettuce on the Bread*). Add a `HasObjectClass` Predictate using the class name you are looking for from your object detector classes (e.g., `Bread`). 
6. Continuing adding states and transitions until your have completed building the sandwich.
7. End the FSM with a `Finished` state and a transition from the last task state.

Your completed FSM will look something like this:
![image](https://github.com/cmusatyalab/wcacourse/assets/6760112/e883bb14-8640-4a79-9be0-10226df94143)


Now, now `Export` the FSM to your laptop. The `app.pbfsm` file will be used in the following steps.

### *Pro Tips*
* States are defined from the *system's* point of view, not the *user's*. That means that a state is typically *"looking for the next object"* rather than *"waiting for the user to do something"*.
* Optionally, add an `Image Instruction` to you transitions.

## Install the FSM and object detector in your WCA Backend

Upload the `app.pbfsm` file to the cloudlet. From your laptop:
```sh
$ scp -i ~/.ssh/wca-student.pem <PATH_TO_app.pbfsm> wcastudent@<CLOUDLET_NAME>:GatingWCA/server/app.pbfsm
```
Login to your backend:
```sh
$ ssh -i ~/.ssh/wca-student.pem wcastudent@<CLOUDLET_NAME>
```
Make sure your trained model is in `/home/wcastudent/models`

## Run the WCA Backend
From the cloudlet
```sh
$ cd
$ source venv/bin/activate
$ cd GatingWCA/server
$ python3.8 server.py app.pbfsm
```

## Connect your WCA Client to the Backend
On your smartphone home screen, select the `WCA` icon. Make sure that Wi-Fi is on and you are connected to CMU-SECURE network.

## Team Specific Information

| Team | Laptop Info|Essential Phone|CVAT Username| Cloudlet Name                 | IP Address     |
| ---- | ----|----|----|----|----|
| 1    | soar|  lm1  |lmtraining1  | lmtraining1.livingedgelab.org | 52.21.253.39   |
| 2    | dip|  lm2  |lmtraining2  | lmtraining2.livingedgelab.org | 34.202.133.19  |
| 3    | swoop  |  lm3  |lmtraining3  | lmtraining3.livingedgelab.org | 34.230.218.14  |
| 4    | dive  |lm4  |lmtraining4  | lmtraining4.livingedgelab.org | 174.129.14.131 |



<!--- ## 3rd Day -->
## References
* __Labeling__ - [CVAT](https://github.com/cvat-ai/cvat)
* __Dataset Management__ - [opentpod-tools](https://github.com/cmusatyalab/opentpod-tools/), Datumaro
* __WCA Workflow__ - [OpenWorkflow](https://github.com/cmusatyalab/OpenWorkflow)
* __Backend__ - [GatingWCA Python Server](https://github.com/cmusatyalab/GatingWCA/tree/main/server)
* __Android App Development__ - [GatingWCA Android Application](https://github.com/cmusatyalab/GatingWCA/tree/main/android-client), [Android Studio](https://developer.android.com/studio)
