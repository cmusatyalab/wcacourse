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
<NEED DIRECTIONS> I believe this is done directly from the smartphone
https://cvat.cmusatyalab.org/auth/login

## Annotate the Dataset
Connect your laptop to the cvat server and login.
<NEED DIRECTIONS>

## Transfer the Dataset to the Training System
Download the dataset that you created above in yolo format to your laptop. Is this where opentpod dedup, etc., steps happen? <NEED DIRECTIONS>
<SCREENSHOT>
<NEED DIRECTIONS>

Upload the dataset on the cloudlet. Where?
<NEED DIRECTIONS>

## Train the Object Detector
<NEED DIRECTIONS>
Where does the model end up?

## Define the Application Logic
OpenWorkFlow
[Documentation](https://github.com/cmusatyalab/OpenWorkflow)
[Access the Tool](https://cmusatyalab.github.io/OpenWorkflow/)
<NEED DIRECTIONS>

## Install the FSM and object detector in your WCA Backend
Download the pbfsm file from OpenWorkFlow onto your laptop.
<NEED DIRECTIONS>

Upload the pbfsm file to the cloudlet. From your laptop:
```
scp -i ~/.ssh/wca-student.pem <PATHTOFSMFILE> wcastudent@<DOMAIN_NAME>:GatingWCA/server/app.pbfsm
```
Login to your backend:
```
ssh -i ~/.ssh/wca-student.pem <PATHTOFSMFILE> wcastudent@<DOMAIN_NAME>
```
Copy the model into the path defined in your pbfsm file. Try to make it `/home/wcastudent/models`.
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

| Team | Account                  | CLOUDLET NAME                 | IP Address     |
| ---- | ------------------------ | ----------------------------- | -------------- |
| 1    | cmusatyalab001@gmail.com | lmtraining1.livingedgelab.org | 52.21.253.39   |
| 2    | cmusatyalab002@gmail.com | lmtraining2.livingedgelab.org | 34.202.133.19  |
| 3    | cmusatyalab003@gmail.com | lmtraining3.livingedgelab.org | 34.230.218.14  |
| 4    | cmusatyalab004@gmail.com | lmtraining4.livingedgelab.org | 174.129.14.131 |

3rd day
CVAT
Opentpod-tools + ultralytics
Openworkflow
https://github.com/cmusatyalab/OpenWorkflow
Building custom apks for wca app + backend
