[![Build Status](https://travis-ci.org/VoycerAG/gridfs-image-server.svg?branch=master)](https://travis-ci.org/VoycerAG/gridfs-image-server)

Go Image Server
===============

This program is used in order to distribute gridfs files fast with nginx.

It has resizing capabalities and stores resized images in the gridfs filesystem as children
of the original files. 

Compilation:
-----

Install project using ```go get github.com/VoycerAG/gridfs-image-server```


Face Recognition:
----
The image server can provide experimental face detection for all images. In order to use this feature you need to install 
openCV in order for the compilation to succeed. This will disable cross compilation compatibilities, since it makes heave use of cgo.

The current algorithm is pretty unconfident and only selects faces if it is about 90% sure that it will actually improve results. This is to be improved in future releases.
## Dependencies on Linux:
```
libcv-dev libopencv-dev libopencv-contrib-dev libhighgui-dev libopencv-photo-dev libopencv-imgproc-dev libopencv-stitching-dev libopencv-superres-dev libopencv-ts-dev libopencv-videostab-dev 
```

## Dependencies on Mac OS X:
```
brew tap homebrew/science
brew install opencv
```

After you have installed all dependencies you can use ```go get -tags=facedetection github.com/VoycerAG/gridfs-image-server``` to install the server with face detection support.

Instructions
-----
```
  -config string
    	path to the configuration file (default "configuration.json")
  -host string
    	the database host with an optional port, localhost would suffice (default "localhost:27017")
  -license string
    	your newrelic license key in order to enable monitoring
  -port int
    	the server port where we will serve images (default 8000)
	(only with buildtag +facedetection)
  -haarcascade string 
    	haarcascade file path
```

Image Server Configuration
-----

See the [configuration.json](configuration.json) file for examples on how to configure entries for the image server.

Newrelic Monitoring
-----
Simply enter a valid license key on startup, and the image server will be monitored with the plugin GoRelic.
You can find it under Plugins in your account.

Nginx Configuration
-----

The configuration section for your media vhost could look something like this:

    location ^~ /media/ {
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $remote_addr;
         proxy_set_header Host $http_host;
         proxy_pass http://127.0.0.1:8000/mongo_database/;
    }


Now resized images can be retrieved by calling ```/media/filename?size=entry``` where as the original image
is still available with ```/media/filename```.

If an invalid entry was requested, the image server will return the original image instead.

## Changelog

Changes in Version 3:

- A bug that made initial resized image do not sent cache headers is fixed.
- If neither the original file, nor the resize file could be found, instead of a status code 400
the server will now respond with a status code 404.
- If resizing fails, there won't be a status code 400 anymore, instead it will be a status code 500.
- Configurations without resize type are no longer supported. 
- image magick fallback is dead, therfore you should at least use go 1.4 now, 1.5 is recommened.
