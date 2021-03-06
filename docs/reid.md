## Cross-camera Re-Identification

This example shows how to use the deep learning models to do feature extraction.

--------

## How to use on **Linux**:

#### 1. Go to Sunergy and compile
```python
cd Sunergy
make
```

#### 2. Copy libsunergy.so to the cross-camera_re-id folder 

```pyhton
cp -i lib/linux/libsunergy.so example/python/cross-camera_re-id
```

#### 3. Enter the cross-camera_re-id folder 

```python
cd example/python/cross-camera_re-id
```

#### 4. Load image and model 

Check if the following four files' locations and names are consistent with  the following code.


```python
"../../model/extract/re_id.names"
"../../model/extract/re_id.cfg"
"../../model/extract/re_id.weights"
"../../model/extract/test.jpg"
```
&nbsp;&nbsp;**re_id.names** ---Empty file.  
&nbsp;&nbsp;**re_id.cfg** ---The structure of the deep neural network.  
&nbsp;&nbsp;**re_id.weights** --Trained weight.  
&nbsp;&nbsp;**test.jpg** --- The image you want to do feature extraction.

#### 5. Run

```python
python cross-camera_re-id.py
```

-----------

## How to use on **Windows**: 

### C

#### 1. Open project
Start VS2015, open Sunergy.sln, set x64 and Release.

#### 2. Load image and model 

Check if the following four files' locations and names are consistent with  the following code.


```C
	char names[] = "../../model/extract/re_id.names";  
	char cfg_file[] = "../../model/extract/re_id.cfg";
	char weight_file[] = "../../model/extract/re_id.weights";
	char image_file[] = "../../model/extract/test.jpg";
```

&nbsp;&nbsp;**re_id.names** ---Empty file.  
&nbsp;&nbsp;**re_id.cfg** ---The structure of the deep neural network.  
&nbsp;&nbsp;**re_id.weights** ---Trained weight.  
&nbsp;&nbsp;**test.jpg** --- The image you want to do feature extraction.

#### 3. Build Sunergy
Choose project **sunergy**, do the: *Property -> Configuration type -> Static Library(.lib)*.  <br> Then do the: *Build -> Build Sunergy*.
#### 4. Build and run
Choose project **corss-camera_re-id**, do the: *Build -> Build corss-camera_re-id*.   
Set **corss-camera_re-id** as the startup project and run it.


### python

#### 1. Open project
Start VS2015, open Sunergy.sln, set x64 and Release.
#### 2. Build Sunergy
Choose project **sunergy**, do the: *Property -> Configuration type -> Dynamic Library(.dll)*.<br>  Then do the: *Build -> Build Sunergy*.
#### 3. Copy libsunergy.dll to the cross-camera_re-id folder 
Copy the **libsunergy.dll** in *lib/windows* to *example/python/corss-camera_re-id*.    
Rename it as **libsunergy.pyd**.
#### 4. Load image and model 

Check if the following four files' locations and names are consistent with  the following code.

```python
"../../model/extract/re_id.names"
"../../model/extract/re_id.cfg"
"../../model/extract/re_id.weights"
"../../model/extract/test.jpg"
```

&nbsp;&nbsp;**re_id.names** ---Empty file.  
&nbsp;&nbsp;**re_id.cfg** ---The structure of the deep neural network.  
&nbsp;&nbsp;**re_id.weights** ---Trained weight.  
&nbsp;&nbsp;**test.jpg** --- The image you want to do object detection

#### 5. Open cmd and run 
```python
cd example/python/corss-camera_re-id
python corss-camera_re-id.py
```


------

&nbsp;
#### Code:

#### C
```C
#include "Sunergy.h"
#include <stdio.h>
#ifdef _WIN32
#include "io.h"
#else
#include <unistd.h>
#include <fcntl.h>
#endif

#include "opencv2/highgui/highgui_c.h"
#include "opencv2/imgproc/imgproc_c.h"


void set_data(char* file, sunergy_data_t* data)
{
	IplImage* src = 0;
	src = cvLoadImage(file, 1);
	data->width = src->width;
	data->height = src->height;
	data->step = src->width * 3;
	data->batch = 1;
	data->channel = 3;
	data->data_location = CPU_DATA;
	data->format = BGR;
	data->preprocessing = false;
	data->type = DATA_UCHAR;
	data->data = src->imageData;
}


void show_image_extract(char* file, extract_out_t* out)
{
	IplImage* src = cvLoadImage(file, 1);
	cvNamedWindow("extract", CV_WINDOW_AUTOSIZE);
	CvPoint lt, rb;
	CvScalar color;
	
	cvShowImage("extract", src);
	cvWaitKey(0);
	cvReleaseImage(&src);
	cvDestroyWindow("extract");
}


int main(int argc, char** argv)
{
	char names[] = "../../model/extract/re_id.names";
	char cfg_file[] = "../../model/extract/re_id.cfg";
	char weight_file[] = "../../model/extract/re_id.weights";
	char image_file[] = "../../model/extract/test.jpg";
	FILE* fp0, *fp1, *fp2, *fp3;
	fp0 = fopen(names, "r");
	fp1 = fopen(cfg_file, "r");
	fp2 = fopen(weight_file, "r");
	fp3 = fopen(image_file, "r");
	if (fp0 == NULL || fp1 == NULL || fp2 == NULL || fp3 == NULL)
	{
		printf("please run modle.sh\n");
		exit(0);
	}
	int gpuid = 0;
	bool inference = true;
	int ret = 0;
	sunergy_data_t data;
	extract_out_t out;
	net_information_t info;
	sunergy_t sun = sunergy_init(names, cfg_file, weight_file, gpuid, inference);
	sunergy_get_network_infomation(sun, 1, &info);
	set_sunergy_param(sun, 0, 0, 255, 0.25, 768, 576, 1);
	set_data(image_file, &data);
	printf("start\n");

	ret = sunergy_extract(sun, data, &out);

	for (int i = 0; i < out.size; i++)
	{
		printf("%.8f \n",out.feat[i]);
	}
	
	//show_image_extract(image_file, &out);
	ret = sunergy_free(sun);

	system("pause");
}


```


#### python
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import libsunergy as sun
import cv2
import time
import numpy as np

def extract():
    exnet1 = sun.init("../../model/extract/re_id.names",#names file
             "../../model/extract/re_id.cfg",#cfg file
             "../../model/extract/re_id.weights")#weight file
    exnet = exnet1[0]['index']
    nms=0.4 #param of detect
    means=127.5 # image reduce means before run in network
    normalization=128 #image normalization coefficient
    threshold=0.24#detect threshold
    video_w= 640#image real width used by detect and pose
    video_h=424#image real height used by detect and pose
    resize_format = 0 # 0: darknet 1: opencv 2:caffe
    sun.set_param(exnet,nms,means,normalization,threshold,video_w,video_h, resize_format)

    #frame = cv2.imread("/home/al/pytorch2sunergy/reid_sunergy/test.jpg")
    frame = cv2.imread("../../model/extract/test.jpg")
    # cv2.imshow("1",frame)
    frame1 = cv2.resize(frame, (112, 112))
    frame = cv2.cvtColor(frame1, cv2.COLOR_BGR2RGB)
    img={"width":frame.shape[1],#image width
         'height':frame.shape[0],#image height
         'step':frame.shape[2]*frame.shape[1],#image step
         'channel':frame.shape[2],#image channel
         'batch':1,#image batch
         'location':0,#location of image data 0:CPU 1:GPU
         'preprocessing':0,#image data run in network directly,0:image with no preprocessing 1: image have preprocessed
         'format':1,#format of image data 0:RGB 1:BGR 2: RGP 3:BGRP
         'type':1,#type of image data 0: char 1: float
        'data':frame.data}#image data

    ret = sun.extract(exnet,img)
    print ((ret))
    #print np.array(ret)
    #f = open('feature.txt', 'wb')
    #f.write(str(np.array(ret)))
    #f.close()
    sun.free(exnet)

if __name__ == "__main__":
   extract()
```
