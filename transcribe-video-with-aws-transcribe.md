---
layout: layouts/blog.njk
title: How to use AWS Transcribe to transcribe video
name: AWS Transcribe
type: Content
description: Use AWS Transcribe to transcribe a video's audio to text using the AWS CLI or with Node.js.
date: 2021-05-24
author: Joyce Echessa
thumbnail: aws-transcribe-thumb.jpg
tags:
  - how to
  - learn
related:
  - How to add captions to video using PHP
  - Introduction to subtitles
  - AWS Transcription to subtitle
---

This guide is the first part of a two-part guide on how to use AWS Transcribe to transcribe the audio in a video to text.
The second part will cover how you can use your transcribed audio and 
[convert these to captions](/learn/convert-aws-transcriptions-to-srt-subtitles).

### What is Amazon Transcribe?

With more than 80% of videos on social media being watched on mute adding captions to your videos can greatly increase 
how successful they are at communicating your message. Subtitles make a video more accessible to those who might have 
impaired hearing and to those who might find it hard to follow along where the spoken language may not be not be in 
their native tongue. Subtitles can also improve a video's SEO since Google indexes captions that you add to your videos, 
leading to potential boosts boosts in search engine rankings.

To add subtitles to a video, you can transcribe it manually by hand, or you can let technology help you along by
using a speech-to-text service like [Amazon Transcribe](https://aws.amazon.com/transcribe/). Amazon Transcribe uses
machine learning to automatically transcribe speech to text and works with both audio and video files.

### Transcribing a video with AWS transcribe

There are several ways that you can transcribe audio/video with AWS Transcribe: you can use the **Amazon Transcribe
Console**, the **AWS Command Line Interface (AWS CLI)** or one of the various available **SDKs** for your preferred 
language. We'll cover the last two options, but you can check out [this
guide](https://aws.amazon.com/getting-started/hands-on/create-audio-transcript-transcribe/) for instructions on using
the web console.

To get started with Amazon Transcribe, you will first need to set up an [AWS account](https://aws.amazon.com/) and
[create an AWS Identity and Access Management (IAM) user](https://docs.aws.amazon.com/transcribe/latest/dg/setting-up-asc.html#setting-up-asc-iam). 
You can use your AWS account credentials to access the API, but for security reasons, it is highly recommended that 
you access AWS using IAM user credentials.

#### Transcribing a video with the AWS Command Line Interface (AWS CLI)

There are 2 versions of the AWS CLI available for use: Version 1 and Version 2. We'll be using Version 2. If you haven't
done so already, [install the CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) for your
particular OS. After installation, follow the [steps outlined in this document](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) 
to configure AWS CLI with an account access key ID, secret access key, region and default output format. These are 
required for the CLI tool to be able to interact with AWS.

With the AWS CLI set up, we are now ready to transcribe a video. We'll use the video in [this repository]
(https://github.com/shotstack/test-media/tree/main/captioning) for the demo. The video/audio file that you
want to transcribe has to be hosted in an [AWS S3 Bucket](https://aws.amazon.com/s3/), so make sure you create a bucket 
and add your file there first.

To transcribe an audio or video file, you define details of a transcription job in a JSON file that will be posted to
AWS. Below, we define a transcription job in a file labelled `aws-cli-transcription.json`. The `TranscriptionJobName`,
`LanguageCode`, `MediaFormat` and `Media` fields are required fields for any transcription job, but if you have more
requirements for your particular job, you can take a look at all the [options you can set for a StartTranscriptionJob
operation](https://docs.aws.amazon.com/transcribe/latest/dg/API_StartTranscriptionJob.html).

```json
{
  "TranscriptionJobName": "transcription-job-01",
  "LanguageCode": "en-US",
  "MediaFormat": "mp4",
  "Media": {
    "MediaFileUri": "https://my-bucket.s3.us-east-2.amazonaws.com/scott-ko.mp4"
  }
}
```

Note that the value you give to `TranscriptionJobName` has to be unique within your AWS account as you can use this to
retrieve details of the job. Also, note that the region you used when you were setting up the AWS CLI has to be the same
to the region used for the S3 bucket that you uploaded your media file to — `us-east-2` in our example above.

To transcribe the video in the transcription job, post the JSON file to AWS with the following command (replace the
region and file name with your details):

```bash
$ aws transcribe start-transcription-job \
     --region us-east-2 \
     --cli-input-json file://aws-cli-transcription.json
```

On posting the transcription job, you'll get back data regarding the job:

```json
{
    "TranscriptionJob": {
        "TranscriptionJobName": "transcription-job-01",
        "TranscriptionJobStatus": "IN_PROGRESS",
        "LanguageCode": "en-US",
        "MediaFormat": "mp4",
        "Media": {
            "MediaFileUri": "https://my-bucket.s3.us-east-2.amazonaws.com/scott-ko.mp4"
        },
        "StartTime": "2021-04-29T00:36:28.435000+03:00",
        "CreationTime": "2021-04-29T00:36:28.405000+03:00"
    }
}
```

You can get the results of a transcription job with the following:

```bash
$ aws transcribe get-transcription-job \
   --region us-east-2 \
   --transcription-job-name "transcription-job-01"
```

Below is the result from the above call:

```json
{
    "TranscriptionJob": {
        "TranscriptionJobName": "transcription-job-01",
        "TranscriptionJobStatus": "COMPLETED",
        "LanguageCode": "en-US",
        "MediaSampleRateHertz": 44100,
        "MediaFormat": "mp4",
        "Media": {
            "MediaFileUri": "https://my-bucket.s3.us-east-2.amazonaws.com/scott-ko.mp4"
        },
        "Transcript": {
            "TranscriptFileUri": "LONG URL HERE"
        },
        "StartTime": "2021-04-29T00:36:28.435000+03:00",
        "CreationTime": "2021-04-29T00:36:28.405000+03:00",
        "CompletionTime": "2021-04-29T00:36:53.916000+03:00",
        "Settings": {
            "ChannelIdentification": false,
            "ShowAlternatives": false
        }
    }
}
```

The result you get will depend on the status of the job. From the value of `TranscriptionJobStatus`, we see that the job
was completed successfully and therefore we also get a `Transcript.TranscriptFileUri` field with a link to the
transcript file. If the status was `FAILED`, there would have been a `FailureReason` field with information on why the
job failed. The job status can hold the values `QUEUED `, `IN_PROGRESS`, `COMPLETED` or `FAILED`.

On navigating to the URL set for `TranscriptFileUri`, you will get a file named `asrOutput.json` with the results of the
transcription job. The URL has an expiration time, so if you try to use it and get a message letting you know it has
expired, you need to get the job details again. Below is a cut version of the results of our transcription job:

```json
{
  "jobName": "transcription-job-01",
  "accountId": "140587111551",
  "results": {
    "transcripts": [
      {
        "transcript": "Hi, my name is Scott Co. As an entrepreneur. I cannot overstate how important it is these days to use video as a tool to reach your audience, your community and your customers. People connect with stories. And video allows us to be the most authentic we can be in order to tell those stories. And so if you can present in front of a camera and you have the right tools to support and amplify you, you can be unstoppable."
      }
    ],
    "items": [
      {
        "start_time": "0.54",
        "end_time": "0.95",
        "alternatives": [{ "confidence": "1.0", "content": "Hi" }],
        "type": "pronunciation"
      },
      {
        "alternatives": [{ "confidence": "0.0", "content": "," }],
        "type": "punctuation"
      },
      
      ...
      {
        "start_time": "24.46",
        "end_time": "25.35",
        "alternatives": [{ "confidence": "1.0", "content": "unstoppable" }],
        "type": "pronunciation"
      },
      {
        "alternatives": [{ "confidence": "0.0", "content": "." }],
        "type": "punctuation"
      }
    ]
  },
  "status": "COMPLETED"
}
```

Let's take a look at how we can process a transcription job with code instead of running Terminal commands.

#### Transcribing a video in Node.js with the AWS SDK for JavaScript

Amazon Web Services makes available various SDKs that you can use to access the service. We'll use the
[@aws-sdk/client-transcribe](https://www.npmjs.com/package/@aws-sdk/client-transcribe) package which is the AWS SDK for
JavaScript Transcribe Client for Node.js, Browser and React Native.

To get started, create a new Node.js project by running the following command in your chosen project folder:

```bash
$ npm init -y
```

Then install `@aws-sdk/client-transcribe`:

```bash
npm i @aws-sdk/client-transcribe
```

To use the SDK, you need to specify credentials that will be used to restrict the resources that can be accessed by the
SDK. Credentials can be supplied in various ways:

- Loaded from AWS Identity and Access Management (IAM) roles for Amazon EC2
- Loaded from the shared credentials file (`~/.aws/credentials`)
- Loaded from environment variables
- Loaded from a JSON file on disk
- Loaded from other credential-provider classes provided by the JavaScript SDK

If you have used the AWS CLI before and set up credentials with the tool, you will have a `~/.aws/credentials`
(`C:\Users\USER_NAME\.aws\credentials` on Windows) file on your computer that will be used by the SDK. Since we had set
up the CLI in the last section, we already have this file, so we'll use this option. If you prefer supplying credentials
differently, [check the documentation for
instructions](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/setting-credentials-node.html).

Next, create a file in the root of your project folder labelled `index.js` and add the following:

```javascript
// Import the required AWS SDK clients and commands for Node.js
const {
  TranscribeClient,
  StartTranscriptionJobCommand,
  GetTranscriptionJobCommand,
} = require("@aws-sdk/client-transcribe");
// Set the AWS Region
const REGION = "us-east-2";

// Set the parameters
const params = {
  TranscriptionJobName: "transcription-job-02",
  LanguageCode: "en-US",
  MediaFormat: "mp4",
  Media: {
    MediaFileUri:
      "https://my-bucket.s3.us-east-2.amazonaws.com/scott-ko.mp4",
  },
};

// Create an Amazon Transcribe service client object
const client = new TranscribeClient({ region: "us-east-2" });

const startTranscription = async () => {
  try {
    const data = await client.send(new StartTranscriptionJobCommand(params));
    console.log("Success - StartTranscriptionJobCommand", data);
    getTranscriptionDetails();
  } catch (err) {
    console.log("Error", err);
  }
};

const getTranscriptionDetails = async () => {
  try {
    const data = await client.send(new GetTranscriptionJobCommand(params));
    const status = data.TranscriptionJob.TranscriptionJobStatus;
    if (status === "COMPLETED") {
      console.log("URL:", data.TranscriptionJob.Transcript.TranscriptFileUri);
    } else if (status === "FAILED") {
      console.log("Failed:", data.TranscriptionJob.FailureReason);
    } else {
      console.log("In Progress...");
      getTranscriptionDetails();
    }
  } catch (err) {
    console.log("Error", err);
  }
};

startTranscription();
```

Run the file with:

```bash
$ node index.js
```

We set some parameters for a transcription job and provide values for `TranscriptionJobName`, `LanguageCode`,
`MediaFormat` and `Media`, which are the minimum requirements you need to provide for a job. Check the documentation for
more [options you can set](https://docs.aws.amazon.com/transcribe/latest/dg/API_StartTranscriptionJob.html).

We create an Amazon Transcribe service client object and start the transcription job by using the `GetTranscriptionJobCommand` 
command. After starting a job, we call the `getTranscriptionDetails()` function which uses the `GetTranscriptionJobCommand` 
command to get the details of a particular job. For simplicity, we pass it the `params` object that we had created, but it 
only needs an object with the `TranscriptionJobName` field, so `{TranscriptionJobName: "transcription-job-02 }` would have 
done just fine.

`getTranscriptionDetails()` keeps checking for the status of the job, and while the status is `QUEUED ` or `IN_PROGRESS`, it
will call itself to check the status again. It will keep looping through this cycle until the status turns to `COMPLETED` or 
`FAILED`. If the job completes successfully, it will log the value of `Transcript.TranscriptFileUri` to you console, which links 
to an `asrOutput.json` file that has the complete transcription of the job.

### Using your AWS transcription
You now have an AWS transcription file with the whole transcription of your video. To convert this JSON specification to the 
industry standard SRT [subtitles](https://shotstack.io/learn/introduction-to-captions-subtitles/) format which are able to be
used by all most media players we will need to convert this file.

To learn how to do this take a look at the second part of this guide; 
[Convert AWS Transcriptions to SRT Subtitles](/learn/convert-aws-transcriptions-to-srt-subtitles).