# NexPlayer with AWS Media Services - Integration Guide

In order to use NexPlayer’s streaming capabilities in your NexPlayer
project, you will need to have a video player stack which can stream in
formats compatible with your target devices. In this guide we will set up a
stream for playback on your device. We will use a few different services for
this, including a subset of Amazon Web Services.

We will start by using a simulated live stream.

Amazon S3 -> AWS Elemental MediaLive -> AWS Lambda->
AWS Elemental MediaPackage -> AWS Elemental MediaTailor (optional If Serving Ads) -> Amazon Cloudfront -> NexPlayer

We will then set up a true live stream by swapping S3 for an Encoder.

Open Broadcaster Software (OBS) -> AWS Elemental MediaLive
-> AWS Lambda -> AWS Elemental MediaPackage -> AWS Elemental
MediaTailor [optional If Serving Ads] -> Amazon Cloudfront-> NexPlayer

The key components are:

1. Simulated live stream support. We will use a video file hosted in Amazon **S3**.
2. Live content conversion support. We will use AWS Elemental **MediaLive** in this guide.
3. Distribution creation support. We will use AWS Elemental **MediaPackage** in this guide.
4. Video content delivery support. We will use Amazon **CloudFront** in this guide.
5. In order to insert ads, your content will need SCTE markers. We will use AWS **Lambda** to insert ad markers.
6. Ad decision server support. We will make a static VAST response XML ad decision server **ADS**.
7. Ad insertion support. We will use AWS Elemental **MediaTailor** in this guide.
8. Function scheduling support. We will use Amazon **CloudWatch** in this guide.
9. A capable device. Currently NexPlayer is supported on many device including **iOS,Android, PlayStation, Xbox, WebOS,Tizen** and more!
10. Encoder support. We will create a live stream using the free and open source tool Open Broadcaster Software **OBS** in this guide.

### Live Stream with Amazon S3

#### Stream from S3

The first thing you will need is a live stream. In this guide we’re going to begin
by streaming directly from a video file within S3.This guide assumes that you
have already worked through the VOD Static File guide where we add a video
file to Amazon S3.

In the previous guide, we accessed our file using CloudFront. As the only
system that needs to access our file is AWS Elemental MediaLive, we don’t
need edge caching. So we are going to create a new bucket with a public
video file.

1. Start by navigating to S3 in your AWS console. To keep from conflicting
    with other applications, we recommend you create a new bucket for this
    guide.
	
	- Your bucket will need to have a unique name.		- nexplayer-integration-guide-public
       
	- Your bucket will need to be public
		- Uncheck “Block all public access”

2. Upload one of your videos to the inputs folder within your new S3
    bucket.
	- You are able to specify for the video to loop later in this process, so you don’t need to worry about the video length.
   - 1 minute or longer should be fine.
- Once the video is uploaded, navigate to it and make it public.
	- Click the checkbox next to the video
- Click Actions
- Make Public
- Now click on the object and retrieve its object url

#### AWS Elemental MediaLive - MP4 Input

At this point, we encourage you to skip this section and go to the AWS
Elemental MediaPackage setup. We’ve left the guide in this order for clarity on
the flow of data between systems.

Now, Let’s get started with [AWS Elemental MediaLive](https://console.aws.amazon.com/medialive/home).

1. Navigate to your AWS Elemental MediaLive console
    - Click “ _Create channel_ ”.
    - Give the channel a name.
       - NexPlayerIntegrationGuideS3Channel
    - Specify the IAM Role
       - If no such role exists, create the role from a template and
          then use that role.
    - Specify the Channel class
       - SINGLE_PIPELINE
    - Under Input Attachments click Add
       - Click Create input
          1. Input type: MP4
          2. Input Class: Use a SINGLE_INPUT
          3. Input name:
             NexPlayerIntegrationGuideLiveInputMP4
          4. URL: The object URL from your video in S3
		- After you’ve created the input, set it to loop so that you
			always have content.
			- Source End Behavior: Loop
    - Attach Input.
       - Select the input you just created
		- Confirm.
    - Create an Output
       - Chose MediaPackage as the output group
          1. (If you have not already done so) Go to the AWS Elemental MediaPackage section to create a channel.
          2. Select the AWS Elemental MediaPackage channel ID you created within AWS Elemental MediaPackage
          3. Confirm
				- Got to your AWS Elemental MediaPackage group outputs.
			1. Specify width and height as 1920 and 1080.
			2. Codec settings: H
			3. Aspect Ratio: Specified
			4. Framerate: Specified
    			- Numerator 60
    			- Denominator 1
	- Create Channel
	- Start Channel

#### Inserting Ad Markers - Optional

##### AWS Lambda

With Live Streaming, it’s often unknown ahead of time when you would need
to insert ads. For this project, we will be inserting Ad Markers
programmatically using a combination of AWS Lambda and AWS Elemental
MediaLive. Here is a [link](https://github.com/aws-samples/aws-elemental-mediatailor-tools/tree/master/Workshops/InsertingAdMarkers#2-create-and-test-the-lambda-that-will-insert-ad-markers-to-medialive ) to the source material for this part of the guide
provided by a Media Solutions Engineer at AWS.

1. Navigate to your AWS Lambda console
2. Click “Create function”.
    - Function name: InsertAdMarker
    - Runtime: Python 3.
    - Permissions: Create a new role
       - Attach the MediaLive full access policy to this role
    - Replace the code in lambda_function.py with the code from the
       linked tutorial.
    - Make sure to replace the channel value with your own AWS
       Elemental MediaLive channel ID.
    - Deploy the Lambda.

##### Amazon CloudWatch

Now that you have created a Lambda function, it’s time to schedule that
lambda function to execute on a regular interval. This part of the guide is also
derived from this [tutorial](https://github.com/aws-samples/aws-elemental-mediatailor-tools/tree/master/Workshops/InsertingAdMarkers#3-trigger-lambda-via-cloudwatch-scheduled-events).

We will use CloudWatch Event to Create a rule to invoke your Lambda
function every 1 minute.

1. Navigate to your Amazon CloudWatch console.
2. Under Events, click on “ _Rules_ ”.
3. Click on the “ _Create Rule_ ” button.
4. Under Event Source, choose Schedule.
5. In Fixed rate of, edit to say 1 minute.
6. Under Targets, click on “Add Target”.
7. In the Function dropdown, select your InsertAdMarker function.
8. Click on the “ _Configure details_ ” button.
9. Provide a name like “InsertAdMarkerEveryMinute”.
    - Make sure to leave the State as Enabled. Otherwise, this rule will not run.
    - Click on the “ _Create Rule_ ” button.

With the event enabled, you should see a scheduled Scte35 splice insert in
the MediaLive channel's Schedule every minute.

#### AWS Elemental MediaPackage

You need to create a channel in [AWS Elemental MediaPackage](https://console.aws.amazon.com/mediapackage/home).Additionally,
you will use an automated feature to automatically setup a CloudFront
distribution for that channel.

1. Under Live content, click Create a new channel
    - Give it an id: NexPlayerIntegrationGuideLiveStreamMP4Channel
    - Give it a description as well.
       - Live stream with MP4
    - To enable CloudFront click the radio button to create a CloudFront distribution for this channel.

2. At this point you can now proceed to the AWS ElementalMediaLive setup.
	- Create a channel
	- Specify a new output group in your AWS Elemental MediaLive channel for the AWS Elemental MediaPackage channel you create.

3. Now you need to create an endpoint.
    - Click “ _Add endpoints_ ”.
    - ID: NexPlayerIntegrationGuideS3MpegDashEndpoint
    - In this example we will create an MPEG-DASH endpoint,but you should know that Media Tailor also works with HLS.
	- Packager setting: DASH-ISO
	- Save


#### Inserting Ads - Optional

##### Ad decision server with Amazon S3

In order to insert ads into your live stream, you will need a way to specify what
ads to serve. That’s what an Ad Decision Server can do. Long term you’ll be
using some kind of ad provider, but for this guide, we will use a simple ADS to
accomplish this.

You can learn more about our Ad Decision Server [here](https://aws.amazon.com/blogs/media/build-your-own-vast-3-0-response-xml-to-test-with-aws-elemental-mediatailor/).

1. To make this system work, you’ll need to create a short ad video.
2. We made a sample video that is 7 seconds long.
    - Upload your video to the public S3 bucket you made previously
    - Once the video is uploaded, navigate to it and make it public.
       - Click the checkbox next to the video
		- Click Actions -> Make Public
3. Now you’ll need an xml file which will simulate your ADS.

- Take the sample xml below and replace the CDATA [video] with a link to your video.
       
```xml
<VAST version="3.0">
		<Ad>
		<InLine>
		<AdSystem>2.0</AdSystem>
		<AdTitle>ad-1</AdTitle>
		<Impression/>
		<Creatives>
		<Creative>
			<Linear>
			<Duration>00:00:07</Duration>
			<MediaFiles>
			<MediaFile delivery="progressive"type="video/mp4" 
			width="1920" height="1080">
				<![CDATA[https://yoururl.mp4]]>
			</MediaFile>
			</MediaFiles>
			</Linear>
		</Creative>
		</Creatives>
		</InLine>
		</Ad>
</VAST>
```

- Upload your xml file to your public S3 bucket.
- Once the xml is uploaded, navigate to it and make it public.
	- Click the checkbox next to the video
	- Click Actions -> Make Public

4.Now you are ready to use your test ADS with MediaTailor.

##### AWS Elemental MediaTailor

Now that you have packaged your file for distribution,let’s learn how to place
ads within the distribution using [AWS Elemental MediaTailor](https://aws.amazon.com/mediatailor/).

1. Click Create configuration
    - Name: NexPlayerIntegrationGuideMP4AdCampaign
    - Content Source: Use the URL from the Endpoint you just created without the {name.mpd} or {name.m3u8} section.
	- The content source needs to contain ad markers.
		1. If you have been following this guide then you added a Lambda function and scheduled it with CloudWatch to insert the markers.
		2. If your video already has ad markers, you can skip that step.
    - Ad decision server: Use the url for the VAST xml file you uploaded.
2. Now you can get your video by taking the MediaTailor url prefix and
    appending index.mpd

#### Your Project

Now that you have a streaming service, you can stream your new video on
your target devices. If you have not yet imported NexPlayer into your project,
please follow our integration guide for your platform.

Use the following URL format to play your CloudFront hosted file.
https://{cloudfronturl}/{videoname}

### Live Stream with OBS

Here you will learn how to use everything you’ve learned to stream live.

#### AWS Elemental and OBS Setup

These steps summarize an AWS Guide. Please refer to the steps in the AWS
Guide: “OBS Studio to AWS Elemental MediaLive to AWS”.

1. Check your computer’s public IP address
	- You need the public IP address (or addresses) from the appliance
		that you are using to send the feed to the AWS Elemental
		MediaLive input.
		- Format your IP address to look like x.x.x.x/
2. Create a channel in AWS Elemental MediaPackage.
    - ID: NexPlayerIntegrationGuideOBSChannel
    - Enable the toggle to create a CloudFront distribution.
    - Click “ _Create_ ”
    - Within your new channel is in AWS Elemental Media add a new
       Endpoint.
		- ID: OBSMediaPackageHLSEndpoint
		- Specify HLS
3. Create an input in AWS Elemental MediaLive.
    - Input name: NexPlayerIntegrationGuideLiveInputOBS
    - Input class: SINGLE_INPUT
    - Application name: live
    - Input Security group
		- Create and use a new security group using your ip address in the format: x.x.x.x/
    - Application instance: mystream
4. Configure the OBS Studio software (“the appliance”).
    - Refer to the steps in the AWS Guide for further details:
       - OBS Studio to AWS Elemental MediaLive to AWS
		- In OBS -> Settings -> Stream
			1. Server
				- Input your URL from Media Package
				- Remove the /<stream_name> at the end of the
    			URL so that the server url ends in /live
					- assuming you used “live” for Application
    			name as suggested in this guide
			2. Stream Key
				- Put mystream
					- assuming you used “mystream” for the
    				Application instance name
		- In OBS -> Settings -> Output
			1. Change the video bitrate to 1000 kbps
		- In OBS -> Settings -> Audio
			1. Change the sample rate to 44.1 kHz
		- In OBS -> Settings -> Video
			1. Set the base and output resolution to 1920 x 1080
			2. Change to Fractional FPS Value
    			- Numerator: 30000
    			- Denominator: 1001
5. Create a new channel in AWS Elemental MediaLive.
	- Input name:
	NexPlayerIntegrationGuideOBSHttpLiveStreamChannel
	- Input class: SINGLE_INPUT
	- Application name: live
	- Application instance: mystream
	- Start the video stream.
	- Add Endpoint:
	NexPlayerIntegrationGuideOBSHttpLiveStreamEndpoint
	- Now you can watch the live stream within NexPlayer.

Now it’s time to start the stream.

1. In AWS Elemental MediaLive, on the Channels page,choose the radio
    button next to your new channel.
    - The buttons along the top are enabled.
2. Choose Start.
    - The channel state changes to Starting and then to Running.
3. Switch to OBS and start the stream connection.
    - Video should begin streaming from the appliance

#### Your Project

Now that you have a live streaming service, you can stream your new video
on your target devices. If you have not yet imported NexPlayer into your
project, please follow our integration guide for your platform.

Use the following URL format to play your CloudFront hosted file.
https://{cloudfronturl}/{videoname}

#### Conclusion

Thank you for going through our guide on how to setup Live Streams for your
assets using AWS Media Services and NexPlayer.
