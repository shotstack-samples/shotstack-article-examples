---
layout: layouts/blog.njk
title: Turn images into slideshow videos using Python
name: Turn images into a slideshow videos using Python
type: Content
description: Turn images into a slideshow videos using Python and the Shotstack cloud video editing API.
date: 2022-09-08
author: Kushal Magar
thumbnail: python-slideshow-thumb.jpg
tags:
  - how to
  - learn
related:
  - Convert MP4 to GIF using Python
  - How to create video slideshows with Zapier using Google Forms and YouTube
  - Automatically turn images into a slideshow video using Node.js
---

Slideshow videos are a fantastic way to quickly turn boring images into an exciting piece of content. You compile
images, add some music, some effects and you get an awesome video in minutes. It is popularly used for various use cases
like marketing videos, invitation cards, celebration videos, and more.

Creating a slideshow video is simple and straightforward with desktop and mobile based video editors. However, it gets
repetitive and manual if you need to create more than one video. That's where using Python to programmatically turn
images into videos can be powerful and save you hours of work. After all, we Python users love to automate everything,
don't we?

In this article, you will learn to turn images into a slideshow video with background music using Python.

### Creating a simple slideshow video using Python

At the end of this tutorial, you will generate the following slideshow video using Python:

<video id="player" playsinline controls>
  <source src="https://cdn.shotstack.io/au/stage/c9npc4w5c4/a4199fa0-d65c-42f2-aa5c-c5722fb48886.mp4" type="video/mp4" />
</video>

Let's get started!

### The Shotstack API and Python SDK

Shotstack is a cloud based video editing API that makes it possible to render multiple videos at once. Rendering videos
is CPU intensive and it can take hours to edit and generate videos at scale. With Shotstack you can automate the process
and programmatically generate videos. You can sign up for a
[free developer account](https://dashboard.shotstack.io/register) and get the API key you will need for this tutorial.

You will also need the Shotstack [Python video editing SDK](https://shotstack.io/product/sdk/python/) which uses Python


### Install the SDK and set up credentials

Install the the Shotstack Python SDK using the command line:

```bash
pip install shotstack_sdk
```

You may need to use `pip3` depending on how you configured your Python environment.

Then, set your API key as an environment variable (Linux/Mac):

```bash
export SHOTSTACK_KEY=your_key_here
```

or, if using Windows:

```bash
set SHOTSTACK_KEY=your_key_here
```

Replace `your_key_here` with your provided sandbox API key which is free for testing and development.

#### Importing required modules

This tutorial will create a script to be run from the command line, so lets create an empty file. You can call it what
you like, but for this tuorial lets go with **slideshow.py**.

Open the file and ad some code to import the required modules for the project. We will be using Shotstack Python SDK to
edit and render our video. You can check the [README](https://github.com/shotstack/shotstack-sdk-python) for full
documentation if you first want to learn about the SDK.

```python
import shotstack_sdk as shotstack
import os

from shotstack_sdk.model.soundtrack  import Soundtrack
from shotstack_sdk.model.image_asset import ImageAsset
from shotstack_sdk.api               import edit_api
from shotstack_sdk.model.clip        import Clip
from shotstack_sdk.model.track       import Track
from shotstack_sdk.model.timeline    import Timeline
from shotstack_sdk.model.output      import Output
from shotstack_sdk.model.edit        import Edit
```

### Configuring the API client

Next, set up the client with the API URL and key. It should use the key you added to the environment variables in the
previous step:

```python
host = "https://api.shotstack.io/stage"
configuration = shotstack.Configuration(host = host)
configuration.api_key['DeveloperKey'] = os.getenv('SHOTSTACK_KEY')

with shotstack.ApiClient(configuration) as api_client:
    api_instance = edit_api.EditApi(api_client)
```

### Adding images for the slideshow

We now want to define an array of static images to use in our slideshow. The images need to be hosted online
and be accessible via a public or signed URL. We will be using the following real estate images from the Pexels stock
image library, hosted in our own S3 bucket. You can replace it with your own image urls, as many as you like.

```python
images = [
    "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate1.jpg",
    "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate2.jpg",
    "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate3.jpg",
    "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate4.jpg",
    "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate5.jpg"
]
```

### Combining images to create a video

We no want to loop through the array of photos and stitch them together to create our video. We will iterate over the
`images` array and create [clips](https://shotstack.io/docs/api/#tocs_clip), defining the start time, length, and
motion effect.

We use the `ImageAsset` model to set the image URL and the `Clip` model to create the clip playback properties and add
them to our `clips` array:

```python
clips  = []
start  = 0.0
length = 3.0

for image in images:
    imageAsset = ImageAsset(src = image)

    clip    = Clip(
        asset   = imageAsset,
        start   = start,
        length  = length,
        effect  = "zoomIn"
    )

    start = start + length
    clips.append(clip)
```

Slides begin playing immediately after the previous one finishes. For the first image we default the `start` to **0** so
it starts playing right away. Each image will appear in the video for a duration of **3 seconds**.

The `zoomIn` effect gives a motion effect to all the images. Motion effects you can use to enhance
your video slideshow include:

* `zoomIn` \- slow zoom in
* `zoomOut` \- slow zoom out
* `slideLeft` \- slow slide (pan) left
* `slideRight` \- slow slide (pan) right
* `slideUp` \- slow slide (pan) up
* `slideDown` \- slow slide (pan) down

You can also use slow and fast variants to create faster or slower motion, like: `zoomInSlow` or `slideUpFast`.
### Adding the clips to a tracks

The Shotstack API takes its inspiration from desktop editing software such as; a timeline, tracks, and
clips. Let's create a [track[(https://shotstack.io/docs/api/#tocs_track) and add our array of clips. A track is a
container for clips which works like a layer, you can layer multiple tracks on top of one another.

```python
track = Track(clips = clips)
```

### Adding a soundtrack

To bring our video to life we'll want to add a [soundtrack](https://shotstack.io/docs/api/#tocs_soundtrack). We use the
SDK's `Soundtrack` model to set an audio file URL and a `fadeInFadeOut` volume effect. Similar to images, you can add
the URL of an mp3 file. We will use one from Pixabay's stock library.

```python
soundtrack = Soundtrack(
    src = "https://cdn.pixabay.com/audio/2022/03/23/audio_07b2a04be3.mp3",
    effect = "fadeInFadeOut"
)
```

### Add everything to the timeline

The next step in setting up our edit is to add everything to the timeline 
[timeline](https://shotstack.io/docs/api/#tocs_timeline). The timeline is a container for multiple track arrays, the
soundtrack, plus a setting for the videos background color:

```python
timeline = Timeline(
    background = "#000000",
    soundtrack = soundtrack,
    tracks = [track]
)
```

### Configuring the output and final edit

Last, we need to configure the [output](https://shotstack.io/docs/api/#tocs_output) format and add the timeline and output to
create an [edit](https://shotstack.io/docs/api/#tocs_edit). We use the `Output`  and `Edit` models:

```python
output = Output(
    format      = "mp4",
    resolution  = "sd"
)

edit = Edit(
    timeline = timeline,
    output   = output
)
```
### Sending the edit to the Shotstack API

Finally, we send the data to the [video editing API](/product/video-editing-api/) for processing and rendering. The
Shotstack SDK converts everything we configured using the SDK models to JSON, adds our key to the request header,
and sends everything to the render endpoint.

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

### Final script

In case you skipped the walkthrough, here is the final code in its entirety with a few defensive checks and
optimizations:

```python
import shotstack_sdk as shotstack
import os

from shotstack_sdk.model.soundtrack  import Soundtrack
from shotstack_sdk.model.image_asset import ImageAsset
from shotstack_sdk.api               import edit_api
from shotstack_sdk.model.clip        import Clip
from shotstack_sdk.model.track       import Track
from shotstack_sdk.model.timeline    import Timeline
from shotstack_sdk.model.output      import Output
from shotstack_sdk.model.edit        import Edit

if __name__ == "__main__":
    host = "https://api.shotstack.io/stage"

    if os.getenv("SHOTSTACK_HOST") is not None:
        host =  os.getenv("SHOTSTACK_HOST")

    configuration = shotstack.Configuration(host = host)

    if os.getenv('SHOTSTACK_KEY') is None:
        sys.exit("API Key is required. Set using: export SHOTSTACK_KEY=your_key_here")  

    configuration.api_key['DeveloperKey'] = os.getenv('SHOTSTACK_KEY')

    with shotstack.ApiClient(configuration) as api_client:
        api_instance = edit_api.EditApi(api_client)

        images = [
            "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate1.jpg",
            "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate2.jpg",
            "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate3.jpg",
            "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate4.jpg",
            "https://shotstack-assets.s3.ap-southeast-2.amazonaws.com/images/realestate5.jpg"
        ]

        clips  = []
        start  = 0.0
        length = 3.0

        for image in images:
            imageAsset = ImageAsset(src = image)

            clip    = Clip(
                asset   = imageAsset,
                start   = start,
                length  = length,
                effect  = "zoomIn"
            )

            start = start + length
            clips.append(clip)

        track = Track(clips = clips)

        soundtrack = Soundtrack(
            src     = "https://cdn.pixabay.com /audio/2022/03/23/audio_07b2a04be3.mp3",
            effect  = "fadeInFadeOut"
        )

        timeline = Timeline(
            background = "#000000",
            soundtrack = soundtrack,
            tracks     = [track]
        )

        output = Output(
            format      = "mp4",
            resolution  = "sd"
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

### Running the script

Run the script using python:

```bash
python slideshow.py
```

You may need to use `python3` instead of `python` depending on your configuration.

The script will display the render id returned by the API. We can use the id to retrieve the status of the render.

### Checking the render status and output URL

The render process takes place in the background and may take several seconds. We need another short script that will
check the render status endpoint.

Create a file called **status.py** and paste the following:

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

Then run the script using:

```bash
python status.py {renderId}
```

Replace `{renderId}` with the ID returned from the **slideshow.py** script.

Re-run the status.py script every 4-5 seconds until the status is **done** and a URL is returned. If something
goes wrong the status will show as **failed**.

If everything ran successfully you should now have the URL of the final video, just like the one at the start of the
tutorial.

### Accessing your rendered videos in the dashboard

You can view your rendered videos inside the Shotstack dashboard under **Renders**. Videos are deleted
after 24 hours and need to be transferred to your own storage provider. All files are however copied to 
[Shotstack hosting](https://shotstack.io/docs/guide/serving-assets/destinations/shotstack) and you can configure other
destinations including [S3](https://aws.amazon.com/s3/) and
[Mux]((https://shotstack.io/docs/guide/serving-assets/destinations/mux)).

![Shotstack Dashboard](/assets/img/learn/articles/renders-screenshot.png)

### Final thoughts

This tutorial should have given you a basic understanding of how to programmatically edit and create slideshow videos
using Python and Shotstack video editing API. You could enhance the video with other assets like text, overlay
animations, luma mattes and other features available in the [API reference](https://shotstack.io/docs/api/) docs.

Hopefully this article give you a taste of how to create a fully automated video editing process for different video use
cases like [real estate](http://localhost:8080/solutions/industries/real-estate),
[automotive](http://localhost:8080/solutions/industries/automotive), marketing, sports highlights, and more.
