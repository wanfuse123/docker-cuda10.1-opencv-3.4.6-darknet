# docker-cuda10.1-opencv-3.4.6-darknet
InstallScript and Dockerfile for building Cuda10.1 OpenCV-3.4.6 and Darknet also a second one that just installs CudaLess-OpenCV-3.4.6 and Darknet

Install Ubuntu 18.04 base no need to install full system (just basic desktop) on your host (not in a virtual machine)


create a directory
mkdir ~/copyout/
cd ~/copyout/
Copy the following text to a file called all-darknet-download.sh in that directory

___ begin file ----



#!/bin/bash
wget http://pjreddie.com/media/files/yolo.weights 

wget http://pjreddie.com/media/files/yolo9000.weights

wget https://pjreddie.com/media/files/yolov2-tiny-voc.weights
 	
wget http://pjreddie.com/media/files/vgg-conv.weights 

wget http://pjreddie.com/media/files/shakespeare.weights
wget http://pjreddie.com/media/files/grrm.weights
wget http://pjreddie.com/media/files/tolstoy.weights 
wget http://pjreddie.com/media/files/kant.weights

wget https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt 

wget pjreddie.com/media/files/go.weights 



---- end file ---

execute chmod u+x all-darknet-download.sh

run it with:

./all-darknet-download.sh in directory where you created the all-darknet-download.sh

wait for files to all download


install nividia-driver-430 (latest nvidia driver supporting 10.1 branch)
to do this in Ubuntu goto 

Activities-type in "updates" in search box
click on software & updates icon listed

when it opens go to the Additional Drivers tab

Install nvidia-driver-430 (this is the one that support nvidia and cuda 10.1 (otherwise adjustements to dockerfile are necessary

reboot the system

install docker then invidia runtime 2

follow these directions

docker: 
 (this is a basic way of installing )
 https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04

nvidia-runtime2:
https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)

once those are installed you should be able to run the examples they give (don't proceed further till you do)

note: nvidia-runtim2 is NOT necessary if you are using CPU only version (it runs like crap on the CPU version though)

NEXT:
create a folder

mkdir ~/docker-opencv-cuda
mkdir ~/docker-opencv-nocuda

in the docker-opencv-cuda directory perform a git clone of this repository

git clone https://github.com/wanfuse123/docker-cuda10.1-opencv-3.4.6-darknet.git

do the same in the docker-opencv-nocuda directory

If you wanted the cuda version then in docker-opencv-cuda directory do the following
mv Dockerfile.CUDA Dockerfile

If you want the CPU only version then in the docker-opencv-nocuda directory do the following

mv Dockerfile.CPU Dockerfile

The next step takes about 18 hours on a Core 2 duo with 2 threads

FOR CPU VERSION:
docker build -t nocuda-darknet-opencv-d  .

FOR NVIDIA/CUDA version
docker build -t cuda-darknet-opencv-d .

once they are built you can do the following (hopefully no errors--I believe I have resolved all of them!)

xhost + local:docker && nvidia-docker run -it --name darknet -e DISPLAY="$DISPLAY" -e QT_X11_NO_MITSHM=1 -v /tmp/.X11-unix:/tmp/.X11-unix:rw --privileged -v /home/steven/copyout:/home/docker/copyout -v /dev/video0:/dev/video0 mysnapshot  bash

--name darknet(a unique name each time its run)
mysnapshot is found from 
docker images command output

It will be (if you followed these directions properly)

cuda-darknet-opencv-d or
nocuda-darknet-opencv 

this will put you inside the docker container where you can run

./darknet detector demo data/coco.data cfg/yolov3.cfg yolov3.weights -c 0

if it failes to find a file when you execute it such as a yolov3.cfg or a weights file like yolov3.weights
then do a 
find / -iname yolov3.cfg or find / -iname yolov3.weights
then follow this by a copy
cp <location found> ~/copyout/
  if it doesnt find ~/copyout then try
  cp <location found> /root/darknet/darknet/cfg/
  or 
  cp <location found> /root/darknet/darknet/data/
  
  or 
  cp <location found> /home/docker/darknet/darknet/cfg/
  
  or 
  cp <location found> /home/docker/darkent/darknet/data/

This is likely not to happen since I have added these commands to the dockerfile

lastly
 this : https://github.com/pjreddie/darknet/issues/1052
 
 resolves this: 
 https://github.com/pjreddie/darknet/issues/292 which still may occur even though i have compiled both versions cuda/cpuonly versions this can be caused by whats listed in the 292 link but I have made it so it shouldn't pop up. Its actually caused by the dimensions of the camera video being improper for the NN so placing in the cfg file as described in detail here:
 
 Yolov-4- runs tiny-yolov3 error: darknet: ./src/utils.c:256: error: Assertion `0' fail Aborted (core dumped)
Original

Hy-lscj

2019-03-03 17:38
Running tiny-yolov3 gives an error:

21 Layer before convolutional layer must output image.: No such file or directory 
Darknet: ./src/utils.c:256: error: Assertion `0' failed. 
Aborted (core dumped) 


the reason:
1, the input resize image size is too large, beyond the arithmetic range of the tiny-yolov3 framework

2, the width and height can be changed to a small point (for example, 512 * 512)

THATS it!

I guess there is one more thing

the new way of doing this sort of thing is with tensorflow and tensorflow.js

to learn about tensorflow.js and get a good beginners machine learning tutorial I would suggest this guys videos

https://www.youtube.com/watch?v=Qt3ZABW5lD0&t=19s

or this guys videos

https://www.youtube.com/watch?v=2z0ofe2lpz4

subscribe to these channels and look through the history of the videos they have created and view them in an order that makes sense to what level you are at and that you are comfortable with.

I was starting as a beginner so I started at the beginning.


you will need to install an IDE (like eclipse maybe???) and
perform an install of tensorflow

More directions on this will follow as I learn more.

NOTE: I Have not tried this on OSX or Windows yet. Post something and let me know how it went. Also this docker container is about 18 gig in size (it is monolithic) the process could be slimmed down greatly by using a technique such as
https://imantabrizian.me/posts/2017/06/docker-multistage
https://github.com/NVIDIA/nvidia-docker/issues/806

anyone care to take on the task of making it happen? I don't have the time!

if you do then please fork this repository and send me a message to let me know that you have done this great task!






