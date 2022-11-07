---
layout: layouts/blog.njk
title: How to watermark video using Node.js
name: How to watermark video using Node.js
type: Content
description: Learn how to build a simple application that adds a watermark to a video using a Node.js script.
date: 2020-11-16
author: Derk Zomer
thumbnail: watermark-thumb.jpg
tags:
  - how to
  - learn
related:
  - How to edit a picture-in-picture video using Node.js
  - How to build 1,000 personalised videos
  - Turn images into a slideshow video using Node.js
---

*For anyone not interested in learning how to build an application that watermarks your videos and just wants a simple
way to add a watermark to a video; this [watermarker utility](https://shotstack.io/demo/watermarker/) will do just 
that!*

Watermarking [first appeared](https://en.wikipedia.org/wiki/Watermark) in Italy during the 13th century, where paper 
manufacturers changed the thickness the paper whilst it was still wet. The watermarked part of the paper let through 
more light leading to a mark that allowed readers to identify where the paper was produced.

Today we use watermarks for a wide variety of applications, with the majority of watermarks now being off the digital 
variety. It provides for a clear, yet relatively unobtrusive way, to show original authorship. This is particularly 
important in the age of the internet where it is easy to copy and appropriate media without permission.

This guide has been written to show a quick and easy way to develop an application that can add watermarks to your 
videos using the Shotstack API. This API allows you to describe a video edit in JSON, and then use your favourite 
programming language to render thousands of videos concurrently in the cloud.

### Let's get started

#### Sign up for an API key
You'll need to sign up for a free 
[Shotstack developer account](https://dashboard.shotstack.io/register?utm_source=website&utm_medium=tutorial&utm_campaign=2020_11_tutorial_helloworld). After registering just log in to receive your API key. 

The free version provides you with free use of the API, but does embed a small watermark into your video. You can get 
rid of this by adding in your payment information and using your production key.

#### Node.js
We'll be using [Node.js](https://nodejs.org/en/) to build our application. No fancy routing, just the basics.

### Setting the scene
For this example we'll be pretending we shot some great footage of an open house, and are keen to have this watermarked
so prospective buyers know where to look. We'll use three video clips from Pexels that together will paint a beautiful 
picture of what's for sale:

https://youtu.be/-94YktcVOlI

As our watermark we'll be using the logo of our real estate company; Block real estate:

![A simple watermark](https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/logos/real-estate-black.png)

### The code
For this guide I won't go in too much depth how the API exactly works and what different effects and transitions are
available, but if you're in need of a primer please take a quick look at 
[this guide](https://shotstack.io/learn/hello-world/).

#### JSON
The Shotstack API works by sending a JSON string to the API endpoint. The JSON provides a timeline of clips,
transitions and effects that are converted into an output file such as an MP4 or a GIF.

In the example below we composite our watermark (a PNG file) on top of the three videos. The scale, opacity, position
and offset properties then allow us to position the watermark exactly where we want it to be placed.

```json
{
    "timeline": {
        "background": "#000000",
        "tracks": [
            {
                "clips": [
                    {
                        "asset": {
                            "type": "image",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/logos/real-estate-black.png"
                        },
                        "start": 0,
                        "length": 13,
                        "fit": "none",
                        "scale": 0.33,
                        "opacity": 0.5,
                        "position": "bottomRight",
                        "offset": {
                            "x": -0.04,
                            "y": 0.04
                        }
                    }
                ]
            },
            {
                "clips": [
                    {
                        "asset": {
                            "type": "video",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/footage/suburbia-aerial.mp4"
                        },
                        "start": 0,
                        "length": 5,
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    },
                    {
                        "asset": {
                            "type": "video",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/footage/home-interior-1.mp4"
                        },
                        "start": 4,
                        "length": 5,
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    },
                    {
                        "asset": {
                            "type": "video",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/footage/home-interior-2.mp4"
                        },
                        "start": 8,
                        "length": 5,
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    }
                ]
            }
        ]
    },
    "output": {
        "format": "mp4",
        "resolution": "sd"
    }
}
```

You can submit this JSON to the API using Curl or an application like Postman (see our 
[Hello World tutorial](/learn/hello-world/) on how to use Curl to post directly to the API), but 
for this tutorial we will create a simple application using a Node.js script. Save the above JSON in a file called
`template.json` which our script will read.

#### Node.js application
The Node.js script below takes the JSON, and sends it to the API. It then polls the API to retrieve the render status. 
After about 30 seconds it logs the video URL for you to use. Before running the script you will need to install the 
`dotenv` and `axios` libraries using npm or yarn.

```javascript
require('dotenv').config();
const axios = require('axios');

const shotstackUrl = 'https://api.shotstack.io/stage/';
const shotstackApiKey = process.env.SHOTSTACK_API_KEY; // Either declare your API key in your .env file, or set this variable with your API key right here.

const json = require('./template.json');

/**
 * Post the JSON video edit to the Shotstack API
 * 
 * @param {String} json  The JSON edit read from template.json
 */
const renderVideo = async (json) => {
    const response = await axios({
        method: 'post',
        url: shotstackUrl + 'render',
        headers: {
            'x-api-key': shotstackApiKey,
            'content-type': 'application/json'
        },
        data: json
    });

    return response.data;
}

/**
 * Get the status of the render task from the Shotstack API
 * 
 * @param {String} uuid  The render id of the current video render task 
 */
const pollVideoStatus = async (uuid) => {
    const response = await axios({
        method: 'get',
        url: shotstackUrl + 'render/' + uuid,
        headers: {
            'x-api-key': shotstackApiKey,
            'content-type': 'application/json'
        },
    });

    if (!(response.data.response.status === 'done' || response.data.response.status === 'failed')) {
        setTimeout(() => {
            console.log(response.data.response.status + '...');
            pollVideoStatus(uuid);
        }, 3000);
    } else if (response.data.response.status === 'failed') {
        console.error('Failed with the following error: ' + response.data.response.error);
    } else {
        console.log('Succeeded: ' + response.data.response.url);
    }
}

// Run the script
(async () => {
    try {
        const render = await renderVideo(JSON.stringify(json));
        pollVideoStatus(render.response.id);
    } catch (err) {
        console.error(err);
    }
})();
```

### Initial result
Once we run the Node.js application and the render is complete we should be left with the following video:

https://youtu.be/m_Xy4M_81nw

This is already looking pretty good, but the black watermark in the first scene isn't very clear. It would be helpful
if we can change the watermark we use based on the scene.

### Final touches
We'll add a few final touches such as transitions, a title, some tunes, and switch our watermark image colour based on
the scene.

```json
{
    "timeline": {
        "soundtrack": {
            "src": "https://feeds.soundcloud.com/stream/267703548-unminus-white.mp3",
            "effect": "fadeOut"
        },
        "background": "#000000",
        "tracks": [
            {
                "clips": [
                    {
                        "asset": {
                            "type": "title",
                            "text": "273 Murcheson Drive, East Hampton, NY",
                            "style": "future",
                            "size": "x-small",
                            "position": "bottomLeft",
                            "offset": {
                                "x": 0.6,
                                "y": -0.2
                            }
                        },
                        "start": 1,
                        "length": 5,
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    }
                ]
            },
            {
                "clips": [
                    {
                        "asset": {
                            "type": "image",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/logos/real-estate-white.png"
                        },
                        "start": 0,
                        "length": 5,
                        "fit": "none",
                        "scale": 0.33,
                        "opacity": 0.5,
                        "position": "bottomRight",
                        "offset": {
                            "x": -0.04,
                            "y": 0.04
                        },
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    },
                    {
                        "asset": {
                            "type": "image",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/logos/real-estate-black.png"
                        },
                        "start": 4,
                        "length": 5,
                        "fit": "none",
                        "scale": 0.33,
                        "opacity": 0.5,
                        "position": "bottomRight",
                        "offset": {
                            "x": -0.04,
                            "y": 0.04
                        },
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    },
                    {
                        "asset": {
                            "type": "image",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/logos/real-estate-black.png"
                        },
                        "start": 8,
                        "length": 5,
                        "fit": "none",
                        "scale": 0.33,
                        "opacity": 0.5,
                        "position": "bottomRight",
                        "offset": {
                            "x": -0.04,
                            "y": 0.04
                        },
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    }
                ]
            },
            {
                "clips": [
                    {
                        "asset": {
                            "type": "video",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/footage/suburbia-aerial.mp4"
                        },
                        "start": 0,
                        "length": 5,
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    },
                    {
                        "asset": {
                            "type": "video",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/footage/home-interior-1.mp4"
                        },
                        "start": 4,
                        "length": 5,
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    },
                    {
                        "asset": {
                            "type": "video",
                            "src": "https://shotstack-assets.s3-ap-southeast-2.amazonaws.com/footage/home-interior-2.mp4"
                        },
                        "start": 8,
                        "length": 5,
                        "transition": {
                            "in": "fade",
                            "out": "fade"
                        }
                    }
                ]
            }
        ]
    },
    "output": {
        "format": "mp4",
        "resolution": "sd"
    }
}
```

### Final result
See below our final result — a professionally edited real estate video with watermarks to show off your creation!

https://youtu.be/UIiEw103P1s

### Conclusion
The code in this guide provides for a straightforward way to build an application that takes an image to act as a
watermark and composite it on top of a video.

We've built a more comprehensive [open-source watermarker application](https://shotstack.io/demo/watermarker) which you
can use to watermark your videos, with the
[complete source code](https://github.com/shotstack/watermark-demo?utm_source=website&utm_medium=tutorial&utm_campaign=2020_11_tutorial_watermark)
available on GitHub.

Hopefully, this article has inspired you to start using code to manipulate video. This code could easily be further
adapted to use a video watermark instead of a static image, [use HTML](https://www.youtube.com/watch?v=RUxpAnaUTh4) to
personalise it even further, among many other manipulations at scale that would not be possible using an old fashioned
video editor.

We're always really interested in seeing what people are able to come up with so if you've managed to create something
cool please share!
