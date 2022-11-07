---
layout: layouts/blog.njk
title: Watermark videos using Python
name: Watermark videos using Python
type: Content
description: Watermark videos using Python, Shotstack Python SDK, and the Shotstack video editing API.
date: 2022-10-24
author: Kushal Magar
thumbnail: python-watermark-thumb.png
tags:
  - how to
  - learn
related:
  - Turn images into a slideshow videos using Python
  - How to build 1,000 personalised videos
  - An introduction to Video Automation
---

Watermarking is a common way to label digital content. It enables brands and creators add branding to their content.
Especially in this hyper connected era where videos spread in a blink of an eye, it's important that brands can easily
add branding to their work.

Of course you can use the available video editors or web apps. They sure are simple and easy to watermark a single or a
few videos. But what if you need to watermark thousands of videos? Even better, what if you can build an app that
enables users to watermark videos? 

That's what this tutorial aims to teach - to build a program using Python to automatically watermark videos. After all,
Python developers love to automate, don't they?

This tutorial has two parts:
1. watermarking a single video
2. watermarking multiple videos using a list

### The Shotstack API and SDK

Shotstack provides a cloud based [video editing API](https://shotstack.io/product/video-editing-api/). Rendering videos
is resource intensive and it can take hours to edit and generate videos at scale. Shotstack's rendering infrastructure
makes it possible to build and scale media applications in days, not months.

We will also be using the Shotstack [video editing Python SDK](https://shotstack.io/product/sdk/python/) for this
tutorial. The SDK requires Python 3.

### Install and configure the Shotstack SDK

If you want to skip ahead you can find the source code for this guide in our GitHub
[repository](https://github.com/shotstack-samples/watermark-videos-using-python). Otherwise follow the steps below to
install dependencies and set up your API key.

First of all, install the the Shotstack Python SDK from the command line:

```bash
pip install shotstack_sdk
```
You may need to use `pip3` depending on how your environment is configured.

Then, set your API key as an environment variable (Linux/Mac):

```bash
export SHOTSTACK_KEY=your_key_here
```

or, if using Windows (make sure to add the `SHOTSTACK_KEY` to the path):

```bash
set SHOTSTACK_KEY=your_key_here
```
Replace `your_key_here` with your provided sandbox API key which is free for testing and development.

#### Create a Python script to watermark video
Create a file for the script in your favorite IDE or text editor. You can call it whatever you like, but for this
tutorial we created a file called **watermark-video.py**. Open the file and begin editing.

#### Import the required modules
Let's import the required modules for the project. We need to import modules from the Shotstack SDK to edit and render
our video plus a couple of built in modules:

```python
import shotstack_sdk as shotstack
import os
import sys

from shotstack_sdk.model.clip import Clip
from shotstack_sdk.api import edit_api
from shotstack_sdk.model.track import Track
from shotstack_sdk.model.timeline import Timeline
from shotstack_sdk.model.output import Output
from shotstack_sdk.model.edit import Edit
from shotstack_sdk.model.video_asset import VideoAsset
```

#### Configuring the API client
Next, add the following, which sets up the API client with the API URL and key, this should use the API key added to
your environment variables. If you want, you can hard code the API key here but we recommend using environment
variables.

```python
host = "https://api.shotstack.io/stage"
configuration = shotstack.Configuration(host = host)
configuration.api_key['DeveloperKey'] = os.getenv('SHOTSTACK_KEY')
with shotstack.ApiClient(configuration) as api_client:
    api_instance = edit_api.EditApi(api_client)
```
#### Understanding the timeline architecture
The Shotstack API follows many of the principles of desktop editing software such as the use of a timeline, tracks, and
clips. A [timeline](https://shotstack.io/docs/api/#tocs_timeline) is like a container for multiple clips that includes
different [assets](https://shotstack.io/docs/api/#tocs_asset) which plays over time.
[Tracks](https://shotstack.io/docs/api/#tocs_track) on the timeline allow us to layer clips on top of each other.

#### Setting up the video clip
The video needs to be hosted online and accessible via a public or signed URL. We will use the following 10 second drone
footage as our video asset. You can replace it with your own video url from any online source.

<video id="player" playsinline controls poster="">
  <source src="https://d1uej6xx5jo4cd.cloudfront.net/sydney.mp4" type="video/mp4" />
</video>

Add the following code to create a `VideoAsset` using the video URL:

```python
video_asset = VideoAsset(
    src = "https://d1uej6xx5jo4cd.cloudfront.net/sydney.mp4"
)
```
Next create a `Clip`. A [clip](https://shotstack.io/docs/api/#tocs_clip) is container for different types of 
[assets](https://shotstack.io/docs/api/#tocs_asset), including the `VideoAsset`. We can configure different properties
like length (duration to play for) and start time (when on the timeline the clip will play from). For our clip we use
the `video_asset` we just created, a start of 0 seconds and a length of 10 seconds:

```python
video_clip = Clip(
    asset = video_asset,
    start = 0.0,
    length = 10.0
)
```
#### Setting up the image clip 

Next, we need to add an image which will be the watermark on the video. We will be using the attached logo. You can replace it with your own image url.

![A simple watermark](https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/logos/real-estate-black.png)

 Similar to setting up the `VideoAsset`, let's add the `ImageAsset` by adding the following code.

 ```python
image_asset = ImageAsset(
    src = "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/logos/real-estate-black.png"
)
```

Let's configure the clip properties including the `ImageAsset`. We will configure length, start time, position (position
of the `ImageAsset` in the viewport), scale (size of the asset relative to viewport size), and opacity (transparency of
the asset). Visit the [clip](https://github.com/shotstack/shotstack-sdk-python#clip) documentation to learn more about
clip properties.

```python
image_clip = Clip(
    asset = image_asset,
    start = 0.0,
    length = 10.0,
    scale = 0.25,
    position = 'bottomRight',
    opacity = 0.3
)
```
#### Adding the video clip to the timeline

Next, let's create a [timeline](https://shotstack.io/docs/api/#tocs_timeline) to add our clips. Before that, we need to create two seperate [tracks](https://shotstack.io/docs/api/#tocs_track). We shouldn't add two clips on the same track in the same timeframe as it will not know which asset to show on top. This causes asset to flicker.

Let's add the `image_clip` on `track_1` and the `video_clip` on `track_2` by adding the following script.

```python
track_1 = Track(clips=[image_clip])
track_2 = Track(clips=[video_clip])
```
Next, let's add both tracks on our timeline. Make sure they are ordered sequentially based on how they are layered. If the track consisting the `image_clip` is second on the list, then it won't show on the video as it will stay behind the `video_clip`.

```python
timeline = Timeline(
    background = "#000000",
    tracks = [track_1, track_2]
    )
```

### Configuring the final edit and output
Next, we need to configure our [output](https://shotstack.io/docs/api/#tocs_output). We set the output `format` to
`mp4`. Let's set the video resolution to `hd` which generates a video of 1280px x 720px @ 25fps. You can also configure
other properties like `repeat` for auto-repeat used in gifs, `thumbnail` to generate a thumbnail from a specific point
on the timeline, `destinations` to set export destinations, and more. 

```python
output = Output(
    format = "mp4",
    resolution = "hd"
)

edit = Edit(
    timeline = timeline,
    output   = output
)
```
#### Sending the edit for rendering via the API

Finally, we send the edit for processing and rendering using the API. The Shotstack SDK takes care of converting our
objects to JSON, adding our key to the request header, and POSTing everything to the API.

```python
try:
    api_response = api_instance.post_render(edit)
    message = api_response['response']['message']
    id = api_response['response']['id']
    print(f"{message}\n")
    print(f">> render id: {id}")
except Exception as e:
    print(f"Unable to resolve API call: {e}")
```
#### Final script

The final script is below with a few additional checks and balances:

```python
import shotstack_sdk as shotstack
import os
import sys

from shotstack_sdk.api import edit_api
from shotstack_sdk.model.clip import Clip
from shotstack_sdk.model.track import Track
from shotstack_sdk.model.timeline import Timeline
from shotstack_sdk.model.output import Output
from shotstack_sdk.model.edit import Edit
from shotstack_sdk.model.video_asset import VideoAsset
from shotstack_sdk.model.image_asset import ImageAsset

if __name__ == "__main__":
    host = "https://api.shotstack.io/stage"

    configuration = shotstack.Configuration(host = host)

    configuration.api_key['DeveloperKey'] = os.getenv("SHOTSTACK_KEY")
    
    with shotstack.ApiClient(configuration) as api_client:
        api_instance = edit_api.EditApi(api_client)

        video_asset = VideoAsset(
        src = "https://d1uej6xx5jo4cd.cloudfront.net/sydney.mp4"
        )
        
        video_clip = Clip(
            asset = video_asset,
            start = 0.0,
            length= 10.0
        )
        
        image_asset = ImageAsset(
        src = "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/logos/real-estate-black.png"
        )
        
        image_clip = Clip(
            asset = image_asset,
            start = 0.0,
            length = 10.0,
            scale = 0.25,
            position = 'bottomRight',
            opacity = 0.3
        )
        
        track_1 = Track(clips=[image_clip])
        track_2 = Track(clips=[video_clip])

        timeline = Timeline(
            background = "#000000",
            tracks     = [track_1, track_2]
        )

        output = Output(
            format      = "mp4",
            resolution  = "hd"
        )

        edit = Edit(
            timeline = timeline,
            output   = output
        )

        try:
            api_response = api_instance.post_render(edit)

            message = api_response['response']['message']
            id = api_response['response']['id']
        
            print(f"{message}\n")
            print(f">> render id: {id}")
        except Exception as e:
            print(f"Unable to resolve API call: {e}")
```

#### Running the script
Use the python command to run the script.

```bash
python watermark-video.py
```
You may need to use `python3` instead of `python` depending on your configuration.

If the render request is successful, the API will return the render id which we can use to retrieve the status of the
render.

#### Checking the render status and output URL
To check the status we need another script which will call the API render status endpoint. Create a file called
**status.py** and paste the following code:

```python
import sys
import os
import shotstack_sdk as shotstack

from shotstack_sdk.api import edit_api

if __name__ == "__main__":
    host = "https://api.shotstack.io/stage"
    configuration = shotstack.Configuration(host = host)
    configuration.api_key['DeveloperKey'] = os.getenv("SHOTSTACK_KEY")
    
    with shotstack.ApiClient(configuration) as api_client:
        api_instance = edit_api.EditApi(api_client)
        api_response = api_instance.get_render(sys.argv[1], data=False, merged=True)

        status = api_response['response']['status']

        print(f"Status: {status}")

        if status == "done":
            url = api_response['response']['url']
            print(f">> Asset URL: {url}")
```
Then run the script using command line:

```bash
python status.py {renderId}
```
Replace `{renderId}` with the ID returned from the **watermark-video.py** script. Re-run the status.py script every 4-5
seconds until the status is **done** and a URL is returned. If something goes wrong the status will show as **failed**.
If everything ran successfully you should now have the URL of the final video, just like the one at the start of the
tutorial.

#### Watermarked video example

We can see the watermarked video below:

<video id="player" playsinline controls>
  <source src="https://cdn.shotstack.io/au/v1/62hne3bb81/078c33df-9ac8-461b-9739-6b21f2a61d9b.mp4" type="video/mp4"/>
</video>

#### Accessing your rendered videos using the dashboard

You can view your rendered videos inside the Shotstack dashboard under **Renders**. Videos are deleted after 24 hours
and need to be transferred to your own storage provider. All files are however copied to 
[Shotstack hosting](https://shotstack.io/docs/guide/serving-assets/destinations/shotstack) and you can configure other destinations including [S3](https://aws.amazon.com/s3/) and [Mux](https://shotstack.io/docs/guide/serving-assets/destinations/mux).

![Shotstack Dashboard](/assets/img/learn/articles/render-dashboard-screenshot.png)

### Watermarking multiple videos

As you can see how easy it is to watermark a video using the Shotstack Python SDK. However, the big advantage of using the Shotstack API is how easily we can scale without having to build and manage the rendering infrastructure.

To demonstrate the scalability, we will watermark the following list of videos with the same logo we used above. You can also use other data resources like the CSV and other apps integrating with their APIs.

```python
video_links = [
    'https://d1uej6xx5jo4cd.cloudfront.net/slideshow-with-audio.mp4',
    'https://cdn.shotstack.io/au/v1/msgtwx8iw6/d724e03c-1c4f-4ffa-805a-a47aab70a28f.mp4',
    'https://cdn.shotstack.io/au/v1/msgtwx8iw6/b03c7b50-07f3-4463-992b-f5241ea15c18.mp4',
    'https://cdn.shotstack.io/au/stage/c9npc4w5c4/d2552fc9-f05a-4e89-9749-a87d9a1ae9aa.mp4',
    'https://cdn.shotstack.io/au/v1/msgtwx8iw6/c900a02f-e008-4c37-969f-7c9578279100.mp4'
]
```

The following script watermarks the list of videos. If you want to test it, create a new file called
`watermark-videos.py`, paste the script, and save it. 

```python
import shotstack_sdk as shotstack
import os
import sys

from shotstack_sdk.api import edit_api
from shotstack_sdk.model.clip import Clip
from shotstack_sdk.model.track import Track
from shotstack_sdk.model.timeline import Timeline
from shotstack_sdk.model.output import Output
from shotstack_sdk.model.edit import Edit
from shotstack_sdk.model.video_asset import VideoAsset
from shotstack_sdk.model.image_asset import ImageAsset

if __name__ == "__main__":
    
    host = "https://api.shotstack.io/stage"
    
    configuration = shotstack.Configuration(host = host)

    configuration.api_key['DeveloperKey'] = os.getenv("SHOTSTACK_KEY")
    
    video_links = ['https://d1uej6xx5jo4cd.cloudfront.net/slideshow-with-audio.mp4',
        'https://cdn.shotstack.io/au/v1/msgtwx8iw6/d724e03c-1c4f-4ffa-805a-a47aab70a28f.mp4',
        'https://cdn.shotstack.io/au/v1/msgtwx8iw6/b03c7b50-07f3-4463-992b-f5241ea15c18.mp4',
        'https://cdn.shotstack.io/au/stage/c9npc4w5c4/d2552fc9-f05a-4e89-9749-a87d9a1ae9aa.mp4',
        'https://cdn.shotstack.io/au/v1/msgtwx8iw6/c900a02f-e008-4c37-969f-7c9578279100.mp4'
    ]
    
    with shotstack.ApiClient(configuration) as api_client:
        for link in video_links:
            api_instance = edit_api.EditApi(api_client)

            video_asset = VideoAsset(
            src = link
            )
            
            video_clip = Clip(
                asset = video_asset,
                start = 0.0,
                length= 10.0
            )
            
            image_asset = ImageAsset(
                src = "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/logos/real-estate-black.png"
            )
            
            image_clip = Clip(
                asset = image_asset,
                start = 0.0,
                length = 10.0,
                scale = 0.25,
                position = 'bottomRight',
                opacity = 0.3
            )
            
            track_1 = Track(clips=[image_clip])
            track_2 = Track(clips=[video_clip])

            timeline = Timeline(
                background = "#000000",
                tracks     = [track_1, track_2]
            )

            output = Output(
                format      = "mp4",
                resolution  = "hd"
            )

            edit = Edit(
                timeline = timeline,
                output   = output
            )

            try:
                api_response = api_instance.post_render(edit)

                message = api_response['response']['message']
                id = api_response['response']['id']
            
                print(f"{message}\n")
                print(f">> render id: {id}")
            except Exception as e:
                print(f"Unable to resolve API call: {e}")
```
Then use the python command to run the script.

```bash
python watermark-videos.py
```

To check the render status, run the `status.py` file we created in the first part and run it using the command line:
```bash
python status.py {renderId}
```
Replace the `renderId` from the IDs from returned from the `watermark-videos.py`.

### Final thoughts
This tutorial should have given you a basic understanding of how to programmatically edit and generate videos using
Python and the Shotstack video editing API. As a next step you could learn how to add other assets like text and images
to build a simple media application. This is just an introductory tutorial to programatically working with media but we
can do so much more. We can use this for many use cases like
- [video automation](https://shotstack.io/learn/what-is-video-automation/) 
- [video personalization](https://shotstack.io/learn/what-is-video-personalization/) 
- developing media applications and many more.

You can checkout our other [Python tutorials](http://shotstack.io/learn/how-to/) and [YouTube videos](https://www.youtube.com/c/Shotstack) to learn programmatic media and building video apps faster.