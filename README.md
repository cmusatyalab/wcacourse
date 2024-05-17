# Wearable Cognitive Assistance Student Project Guide
In this class, you will build your own wearable cognitive assistance (WCA) "Sandwich" application. This guide is intended to walk you through the application creation and testing process. At the end of the exercise, you should have a working "toy" application that includes all the current capabilities of CMU's WCA application platform.

Each student will be assigned to a team. The team specific information is provided at the end of this guide. Each team will execute the following steps in creating the application:

1. **Data Collection** -- Use your smart phone to collect videos of the sandwich components and sub-assemblies. These videos will be used in the following two steps to create a training and testing dataset a WCA object detector.
2. **Video Upload** -- Upload your videos to the CVAT annotation tool.
3. **Annotate the Dataset** -- Use CVAT to convert the videos into a labeled yolo dataset.
4. **Transfer the Dataset to the Training System** -- Download the dataset from CVAT and into the cloudlet that will be used for training an object detector.
6. **Train the Object Detector** -- Use the *OpenTPOD* tools to train an object detection neural network model for your application.
7. **Define the Application Logic** -- Use the *OpenWorkFlow* tool to define a finite state machine (FSM) and corresponding user prompts for your application.
8. **Install the FSM and object detector in your WCA Backend** -- Move the application-specifc `*.pbfm` and `*.pt` files into the *GatingFSM* server.
9. **Run the WCA Backend** -- Start the *GatingWCA* server. This server executes your application logi.
10. **Connect your WCA Client to the Backend** -- Launch the client application which will establish a connection to your backend server.

At this point, you are ready and able to actually use your sandwich application. You may find that you need to iterate some or all of these steps in order to get an acceptable detector and tune your FSM.

These steps are described in more detail below. Your development environment consists of three components:
1. A *linux laptop* that you will use to connect to CVAT, OpenTPOD, OpenWorkFlow, and the GatingWCA server.
2. A *GatingWCA server (actually, an AWS VM)* that is preconfigured with the credentials and components you need.
3. An *Android smartphone* that serves as a video data capture device and as the WCA application client.

## Data Collection
Collect a series of X to Y videos of approximately Z length using your smartphone camera. Videos should contain ... <NEED SOME HELP WITH THIS>

## Video Upload
Using the browser on the phone, navigate to [CVAT](https://cvat.cmusatyalab.org/auth/login) and login using your [credentials](https://github.com/cmusatyalab/wcacourse/edit/main/README.md#team-specific-information). Upload the videos you captured in the previous step to a new project.

## Annotate the Dataset
Connect your laptop to the cvat server and login.
<NEED DIRECTIONS>

## Transfer the Dataset to the Training System

SSH to your VM from the laptop and then download and manipulate the dataset using opentpod-tools and datumaro.

```sh
    # upload videos to CVAT, and label them

    # download labeled datasets
    $ tpod-download [--project|--task|--job] <dataset0> .. <datasetN>

    # merge datasets
    $ datum merge [-o merged] <dataset0> .. <datasetN>

    # filter frames with no annotations (and optionally annotated occlusions)
    $ tpod-filter [--filter-occluded] [-o filtered] merged

    # remove duplication
    $ tpod-unique [-m sequential|random|complete] [-o unique] merged [-t 10 -r 0.7]
    # -m --method: sequential, only check against the last 'unique' image
    #              random, check against random subset of unique image list with [-r/--ratio]
    #              exhaustive, check each new image against all chosen unique images
    # -t --threshold: the difference between current image and unique image(s), default = 10

    # split into training and validation subsets
    $ datum transform -t random_split -o split unique -- -s train:0.9 -s val:0.1 [-s test:...]
```


## Train the Object Detector

Train a yolo model using the curated dataset.

```sh
    # export to yolo_ultralytics format
    $ datum convert -i split -f yolo_ultralytics -o yolo-dataset -- --save-media

    # train an object detector model
    $ yolo detect train data=$(pwd)/yolo-dataset/data.yaml model=yolov8n.pt epochs=100 imgsz=640 project=yolo-project
```

## Define the Application Logic
OpenWorkFlow
[Documentation](https://github.com/cmusatyalab/OpenWorkflow)
[Access the Tool](https://cmusatyalab.github.io/OpenWorkflow/)
<NEED DIRECTIONS>

From the OpenWorkFlow, create your application finite state machine:
1. Create a `Start` state.
2. Create your first task state (e.g., `Bread`). Add a `YoloProcessor` to this state. Make sure you add in the conf_threshold. Leaving the default will cause an error. The model_path should be `/home/wcastudent/model/<YOUR_MODEL_FILE>.pt`.
3. Add a transition between the Start state and the Bread state. Enter in an Audio Instruction for the task that the user should do to progress to the Bread state (e.g., *Put the Bread on the Table*). Do not add a predicate.
4. Create your second task state (e.g., `Bread-Lettuce`) with a `YoloProcessor`. Make sure you add in the conf_threshold.
5. Add a transition between the Bread state and the Bread-Lettuce state. Enter in an Audio Instruction for the task that the user should do to progress to the Bread-Lettuce state (e.g., *Put the Lettuce on the Bread*). Add a `HasObjectClass` Predictate. *Is class name the class number or a name?*
6. Continuing adding states and transitions until your have completed building the sandwich.
7. End the FSM with a `Finished`state and a transition from the last task state.

Your completed FSM will look something like this:
![image](https://github.com/cmusatyalab/wcacourse/assets/6760112/8feddc0d-666b-4c0f-bb9a-402852d5f406)

Now, now `Export` the FSM to your laptop. The `app.pbfsm` file will be used in the following steps.

## Install the FSM and object detector in your WCA Backend
<NEED DIRECTIONS>

Upload the `app.pbfsm` file to the cloudlet. From your laptop:
```
scp -i ~/.ssh/wca-student.pem <PATHTOFSMFILE> wcastudent@<DOMAIN_NAME>:GatingWCA/server/app.pbfsm
```
Login to your backend:
```
ssh -i ~/.ssh/wca-student.pem <PATHTOFSMFILE> wcastudent@<DOMAIN_NAME>
```
Copy the model into the path defined in your pbfsm file.
```
cp <TPOD_DOWNLOAD_DIR>/<MODELFILE> /home/wcastudent/models/
```

## Run the WCA Backend
From the cloudlet
```
source venv-opentpod/bin/activate
cd GatingWCA/server
python3 server.py <PBFSMFILE>
```

## Connect your WCA Client to the Backend
<NEED DIRECTION>

## Team Specific Information

| Team | Laptop Info|Essential Phone|CVAT Username|PlayStore Account                  | CLOUDLET NAME                 | IP Address     |
| ---- | ----|---|---------|-------- | ----------------------------- | -------------- |
| 1    | soar|  lm1  |lmtraining1  |cmusatyalab001@gmail.com | lmtraining1.livingedgelab.org | 52.21.253.39   |
| 2    | dip|  lm2  |lmtraining2  |cmusatyalab002@gmail.com | lmtraining2.livingedgelab.org | 34.202.133.19  |
| 3    | swoop  |  lm3  |lmtraining3  |cmusatyalab003@gmail.com | lmtraining3.livingedgelab.org | 34.230.218.14  |
| 4    | dive  |lm4  |lmtraining4  |cmusatyalab004@gmail.com | lmtraining4.livingedgelab.org | 174.129.14.131 |



## 3rd Day

* __Labeling__ - [CVAT](https://github.com/cvat-ai/cvat)
* __Dataset Management__ - [opentpod-tools](https://github.com/cmusatyalab/opentpod-tools/), Datumaro
* __WCA Workflow__ - [OpenWorkflow](https://github.com/cmusatyalab/OpenWorkflow)
* __Backend__ - [GatingWCA]([https://github.com/cmusatyalab/GatingWCA](https://github.com/cmusatyalab/GatingWCA/tree/main/server))
* __Android App Development__ - [GatingWCA](https://github.com/cmusatyalab/GatingWCA/tree/main/android-client), [Android Studio](https://developer.android.com/studio) 
