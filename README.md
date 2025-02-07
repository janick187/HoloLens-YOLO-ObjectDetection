# Hololens YOLOv4 Object Detection
With this repository you can access and evaluate the camera stream of your Microsoft Hololens to recognize at what objects the user is currently looking at. 
The code is largely a combination of two repositories.
- Access video-stream of Hololens PV camera: [IntelligentEdgeHOL](https://github.com/Azure/IntelligentEdgeHOL)
- Run YOLOv4 object detection: [tensorflow-yolov4-tflite](https://github.com/theAIGuysCode/tensorflow-yolov4-tflite)

## Geeting Started

### [1] Device Portal Credentials
[Configure](https://docs.microsoft.com/en-us/windows/mixed-reality/develop/platform-capabilities-and-apis/using-the-windows-device-portal) the Hololens Device Portal. Save and remember [your user credentials](https://docs.microsoft.com/en-us/windows/mixed-reality/develop/platform-capabilities-and-apis/using-the-windows-device-portal#creating-a-username-and-password).

### [2] Clone repository
`git clone https://github.com/Interactions-HSG/21-MT-JanickSpirig-DC-Holo`

### [3] Setup YOLOv4
- Open a terminal window and cd to modules\YoloModule\app\detector
- Create and activate your conda environment by first [downloading anaconda](https://docs.anaconda.com/anaconda/install/index.html) and then executing one of the commands below in terminal (either CPU or GPU)

```
cd modules/YoloModule/app/detector/

# CPU
conda env create -f conda-cpu.yml
conda activate yolov4-cpu

# GPU
conda env create -f conda-gpu.yml
conda activate yolov4-gpu
```

- Download the pre-traiend [coco weights](https://cocodataset.org/#home) ([yolov4.weights](https://drive.google.com/open?id=1cewMfusmPjYWbrnuJRuKhPMwRe_b9PaT), [yolov4-tiny.weights](https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v4_pre/yolov4-tiny.weights)) __OR__ pre-trained [interactions weights](https://drive.google.com/file/d/10xhruakVoIGTGAzH7BTxPX7rrcfWJByo/view?usp=sharing) (office and shopfloor evnironment) __OR__ train yolo for your custom classes by following the steps in [this video](https://www.youtube.com/watch?v=mmj3nxGT2YQ) 
- Move the according `.weights` file (pretrained or custom) into the folder detector\data. If you use custom wieights, make sure that the file is named as `custom.weights`
- If you use custom weights, place your custom `.names` file (e.g. `obj.names`) ([download](https://drive.google.com/file/d/1lttBOLLfv_L71n6GMPmJasXOws_eMzF8/view?usp=sharing) the interactions `.names`) into folder detector\data and comment out line 18 in file detector\core\config.py and comment line 15. It should look like in the screenshot below. Remember that anytime you switch between custom wnd pretrained coco weights (pre-trained) you have to adjust these two lines of code.

<img width="541" alt="Screenshot 2021-08-15 at 15 04 22" src="https://user-images.githubusercontent.com/43849960/129479546-edf3ba64-9743-4e59-96b2-e42444e83af5.png">

- Convert the yolo weights from darkent to TensorFlow format by executing one of the commands below in the terminal
```
cd modules/YoloModule/app

# pretrained
python save_model.py --weights detector/data/yolov4.weights --output detector/checkpoints/yolov4-416 --input_size 416 --model yolov4 

# pretrained tiny
python save_model.py --weights detector/data/yolov4-tiny.weights --output detector/checkpoints/yolov4-tiny-416 --input_size 416 --model yolov4 --tiny

# custom
python save_model.py --weights detector/data/custom.weights --output detector/checkpoints/custom-416 --input_size 416 --model yolov4 
```
### [4] Setup conifg.yml
Define the options in the file `config.yml` according to your needs.  

| Parameter  | Details |
| ------------- | ------------- |
| `CUSTOM`  | Defines wehter YOLOv4 has been trained for custom classes. Set to `TRUE` if you have set up YOLOv4 for custom classes |
| `CUSTOM_CLASSES` | Stores the labels of all custom classes. If `CUSTOM` is set to true, list here all class labels as bullet points (use `-` and isnert line break after each label  |
| `USE_YOLO-TINY` | Defines whether the yolo wieghts are tiny or normal ones. Set to `TRUE` if you are using tiny wieghts |
| `VIDEO_SOURCE` | Defines the URL under which the Hololens camera stream is accessible: `https://<DEVICE-PORTAL-USER>:<DEVICE-PORTAL-PWD>@<HOLOLENS-IP>/api/holographic/stream/live.mp4?olo=true&pv=true&mic=true&loopback=true` The user and pwd are the ones you have defined when setting up the device portal. To come up with the IP address of the Hololens follow [this guide](https://docs.microsoft.com/en-us/windows/mixed-reality/develop/platform-capabilities-and-apis/using-the-windows-device-portal#connecting-over-wi-fi)  |
| `USE_WEBCAM` | Defines whether the camera stream of the Webcam should be used. Set to `TRUE` if you want to run object recognition on your webcam's stream (e.g. to see if YOLOv4 works) |
|`MIN_CONFIDENCE_LEVEL` | Defines the minimum confidence for YOLOv4 in order to label an object with a class name. Set to your desired confidence level (e.g. `0.7`) |
|`MIN_TIME` | Defines the minimum time in seconds that an object must be present in the camera view before the information is sent to the Hololens that a certain object is present. This can be useful to sitinguish between the user just walking around in a room and looking at things randomly and the user looking at a thing on purpose. Set this for example to `1.5` |
| `SHOW_OUTPUT` | Defines whether an output video should be produced under `RESULT_PATH` which contains rectangle boxes and class labels. Set to `True` if you want to double-check the YOLOv4 detections |
|`RESULT_PATH` | Defines the path unser which the result video will be saved (if `SHOW_OUTPUT` is set to `True` |
| `HOLO_ENDPOINT` | Defines wehter and information should be sent to a Hololens or any arbitrary endpoint everytime an object appears in the user's view. Set to `True` if you have set up an endpoint on your Hololens that is able to receive incoming requests. [This example](https://github.com/janick187/Hololens-frontend/blob/master/Assets/Scripts/HTTPListener.cs) shows you how such an endpoint can be setup within a Unity application |
|`HOLO_ENDPOINT_URL`  | Defines the URL and Port of the endpoint (e.g. http://10.2.1.233:5050) to which an HTTP Get request will be sent to whenever a new object has been detected (`http://{HOLO_ENDPOINT_URL}/?{class_label}=1`). Remember to define the port in the URL if necessary |


### [5] Run the detection
Now you are all set and ready to run YOLOv4 object detection on the Microsoft Hololens PV camera. Execute following steps to start the detection.
1. Start up the Hololens and log in. Make sure it is charged sufficiently as the PV camera has heavy battery usage.
2. Activate the conda environment. Open a terminal and execute `conda activate yolov4-cpu` for CPU or `conda activate yolov4-gpu` for GPU.
3. `cd modules/YoloModule/app`
4. `python main.py`
