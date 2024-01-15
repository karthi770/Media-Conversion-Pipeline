# Media Conversion Pipeline

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled.png)

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%201.png)

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%202.png)

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%203.png)

Gave the Administrator Access to the user IAM user karthi so that it can create a role with some permissions while creating a lambda function.

Create a lambda function

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%204.png)

Create a lambda function to convert the media in the S3 bucket into desired format and the converted format shall be stored in destination S3 bucket.

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%205.png)

Create a job.json file which has the code to convert the video into different formats

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%206.png)

The json for the media convertor can be obtained from the create jobs. Enter a test source and destination, give the desired video format and the json for the conversion will be generated which can be used here.
Create a role for Media convert:

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%207.png)

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%208.png)

Go to Lambda and go to configuration tab and select the role.

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%209.png)

Attach the permission to this created role:

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2010.png)

Go to cloud front and create a distribution:

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2011.png)

Make sure the S3 destination bucket is selected.
Then ensure that origin access control settings are created. This will allow to access the objects in S3 only through cloud front.

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2012.png)

Make sure the above selection is done and then create distribution.

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2013.png)

Once the distribution is created, click on the copy policy.

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2014.png)

Paste the copied policy in the S3 destination bucket policy.

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2015.png)

Create an event notification in the S3 source bucket.

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2016.png)

Select the destination as Lambda function and the also select the lambda function that was created.

```bash
import json
import boto3

def lambda_handler(event, context):
    mediaconvert = boto3.client('mediaconvert')
    mediaconvert_endpoint = mediaconvert.describe_endpoints(MaxResults=1)
    mediaconvert = boto3.client('mediaconvert', endpoint_url=f"{mediaconvert_endpoint['Endpoints'][0]['Url']}")
    for message in event['Records']:
				# REPLACE ME #
        destination_bucket = 'iron-man-video-destination'
				##############
        source_bucket = message['s3']['bucket']['name']
        object = message['s3']['object']['key']
        accountid = context.invoked_function_arn.split(":")[4]
        region = context.invoked_function_arn.split(":")[3]

        with open("job.json", "r") as jsonfile:
            job_config = json.load(jsonfile)

        job_config['Queue'] = f"arn:aws:mediaconvert:{region}:{accountid}:queues/ironman_queue"
        job_config['Role'] = f"arn:aws:iam::{accountid}:role/MediaConvert_Default_Role"
        job_config['Settings']['Inputs'][0]['FileInput'] = f"s3://{source_bucket}/{object}"
        job_config['Settings']['OutputGroups'][0]['OutputGroupSettings']['FileGroupSettings']['Destination'] = f"s3://{destination_bucket}/"

        response = mediaconvert.create_job(**job_config)

{
  "Queue": "arn:aws:mediaconvert:region:account_id:queues/Default",
  "UserMetadata": {},
  "Role": "arn:aws:iam::account_id:role/MediaConvert_Default_Role",
  "Settings": {
    "TimecodeConfig": {
      "Source": "ZEROBASED"
    },
    "OutputGroups": [
      {
        "Name": "File Group",
        "Outputs": [
          {
            "ContainerSettings": {
              "Container": "MP4",
              "Mp4Settings": {}
            },
            "VideoDescription": {
              "CodecSettings": {
                "Codec": "H_265",
                "H265Settings": {
                  "MaxBitrate": 5000000,
                  "RateControlMode": "QVBR",
                  "SceneChangeDetect": "TRANSITION_DETECTION"
                }
              }
            },
            "AudioDescriptions": [
              {
                "AudioSourceName": "Audio Selector 1",
                "CodecSettings": {
                  "Codec": "AAC",
                  "AacSettings": {
                    "Bitrate": 96000,
                    "CodingMode": "CODING_MODE_2_0",
                    "SampleRate": 48000
                  }
                }
              }
            ],
            "NameModifier": "-hd"
          },
          {
            "ContainerSettings": {
              "Container": "MP4",
              "Mp4Settings": {}
            },
            "VideoDescription": {
              "Width": 256,
              "Height": 144,
              "CodecSettings": {
                "Codec": "AV1",
                "Av1Settings": {
                  "RateControlMode": "QVBR",
                  "QvbrSettings": {},
                  "MaxBitrate": 100000
                }
              }
            },
            "AudioDescriptions": [
              {
                "CodecSettings": {
                  "Codec": "AAC",
                  "AacSettings": {
                    "Bitrate": 96000,
                    "CodingMode": "CODING_MODE_2_0",
                    "SampleRate": 48000
                  }
                }
              }
            ],
            "NameModifier": "-sd"
          }
        ],
        "OutputGroupSettings": {
          "Type": "FILE_GROUP_SETTINGS",
          "FileGroupSettings": {
            "Destination": "s3://destination_bucket/"
          }
        }
      }
    ],
    "Inputs": [
      {
        "AudioSelectors": {
          "Audio Selector 1": {
            "DefaultSelection": "DEFAULT"
          }
        },
        "VideoSelector": {},
        "TimecodeSource": "ZEROBASED",
        "FileInput": "s3://source_bucket/object.mp4"
      }
    ]
  },
  "AccelerationSettings": {
    "Mode": "DISABLED"
  },
  "StatusUpdateInterval": "SECONDS_60",
  "Priority": 0
}
```

Configure the test event:

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2017.png)

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2018.png)

Change the S3 source bucket name and the S3 object name.

![Untitled](Media%20Conversion%20Pipeline%20edf9bcc247264566980c0539367f0fe5/Untitled%2019.png)

This shows 2 files are created in the destination bucket. With different video definitions.