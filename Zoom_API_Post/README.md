# [Transcribing Zoom Recordings Using the Zoom API and AssemblyAI](https://www.assemblyai.com/blog/transcribing-zoom-recordings-using-the-zoom-api-and-assemblyai)
Link above is the post on AssemblyAI's blog.
Below is my final draft of the same article.

# Transcribing Zoom recordings using the Zoom API and AssemblyAI in Python

![Zoom_90s](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/zoom90s.gif)

Wow, Zoom did itself a huge favor by rebranding since the 90's. This might actually be the main reason they survived the dotcom bubble.

Shame on you if that wry joke is over your head.

Zoom even has an API now folks! One of the many reasons to finally awaken from that 25 year coma! That's right, come back alive to the real simulation, and I'll show you how to use AssemblyAI to transcribe the words spoken in those Zoom meetings that have become such a ubiquitous aspect of our lives.

![matrix](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/matrix.gif)


## Creating a Cloud Recording on Zoom
The first thing to know when creating a cloud recording in Zoom is that you must have a **Pro, Business, or Enterprise account**. The basic free Zoom plan does not have a meeting recording feature enabled. You may already have such a plan, and may already have cloud recordings ready for transcription, in which case, you can skip to the next header and keep rolling.

But if you've just gotten an upgraded Zoom account for your project, and you still need to create a cloud recording, a few things to keep in mind:

- You do not need to have another person on the meeting for cloud recording to work. No need to schedule with someone else, just let it rip on your device and start the recording.

- The button looks like this if you're on a desktop, laptop, or large tablet:
![desktop_button](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/desktop_recording_button.jpg)

- If you're on a phone or small tablet, the cloud recording button hides down yonder behind those 3 dots.
![phone_button](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/phone_button.jpg)

- If you would like to test cloud recording and transcription with multiple people on the line, and you happen to be doing this with a friend's device that's in the same room, a couple of acoustic notes. You'll want to make sure that only one device at a time as unmuted and producing sound, or the feedback will scratch out your eardrums.

- Another strange thing I noticed while cloud recording with multiple devices is that only the non-host's device will feed sound to the recording while you're muting one to prevent feedback. For some reason, if the host's device is unmuted while you're speaking, that sound will not be captured by Zoom's cloud recording.

- You might as well dodge both of those recording dilemmas by having each person and each device in separate rooms. That way the sound barriers prevent the feedback, and both parties can unmute.

Whew, now that those details are out of the way, end that meeting, and let's go find that cloud recording!

Upon ending the recording, a pop-up box will tell you that you'll get an email once the recording is ready. Zoom's help pages cautiously warn users that it could take 24-48 hours for it to be ready, but that was likely the reality in early 2020 when the whole world flocked onto Zoom before their systems were ready for that level of traffic. It may still take a while for longer recordings, but I've found that here in 2021, for the short test recordings I've been transcribing, they are ready just about immediately.

Wow it feels great to talk trash about 2020 like it's a long-gone ex.

Anyway, to find your recording, sign into Zoom, and navigate to **My Account**.
![my_account](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/mtg_my_account.JPG)

To the left, to the left, everything you want in a link to the left. Click on that **"Recordings"** tab over yonder on the left side.
![recordings](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/mtg_recordings.JPG)

And now, the promised land! There you will see your cloud recordings, ready for you in mp4 and m4a format.
![recording_list](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/mtg_recording_list.JPG)


## Accessing Zoom's API Key
Now, let's step through the process of getting keys to Zoom's backdoor. I'd like to give a shoutout to Billy Harmawan for [showing me this somewhat convoluted process](https://medium.com/swlh/how-i-automate-my-church-organisations-zoom-meeting-attendance-reporting-with-python-419dfe7da58c). I'll give you the gist of it here too.

First, you will need to navigate to [Zoom's App Marketplace](https://marketplace.zoom.us/) in a new tab. You may have already been signed in from your main account page, but if not, you can sign in using the same credentials, and then navigate back to the Zoom App Marketplace once logged in.

Second, click **"Develop"** in the upper right corner, and **"Build App"** from the drop-down menu.
![build_app](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/1_app_mkt_build_app.JPG)

Third, you will need to choose **"Create"** under the **"JWT"** app type. We will be using JSON Web Tokens (JWT's) for this transcription connection because of their ease of regeneration, use in code, and appendability to url's.
![app_type](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/2_app_type_jwt.JPG)

Fourth, come up with a unique name for your Zoom app.

![app_name](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/3_app_name.JPG)

Fifth, you will need to fill out the required fields on the information page that it takes you to.
![app_info](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/4_app_info.JPG)

Sixth, on the following page, your API credentials should be generated!
![app_tokens](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/5_zoom_tokens.JPG)

From here, **create a "tokens.env"** text file, and save each credential under a variable name. You will need this .env file in the code later.

![dotenv](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/5_5_dotenv.jpg)

A note: that _first_ JWT token that is generated on the credentials page is nice to have in the dotenv file just in case, but our code will be generating new JWT's that will be used instead. So have it copied, but just know that it might not be necessary.

And lastly, you can skip the optional "Feature" tab on this app creation page and go directly to **"Activation"** to confirm that you are finished.
![active](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/6_app_active.JPG)

These next few steps are just double and triple checks to make sure that the app exists in your account and that the credentials can be managed in the future if you need to.

Navigate back to [Zoom's App Marketplace](https://marketplace.zoom.us/) and click the **"Manage"** button in the upper right corner.
![find_app](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/7_find_app.JPG)

There you will see your app listed.
![dashboard](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/8_app_dashboard.JPG)

**Click on the name of the app**, and when you see this screen once again...
![manage](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/9_manage_app.JPG)

...you can rest assured knowing deep in your bones that the Zoom's API is ready for you.

## Sign up for an AssemblyAI API Token
But are _YOU_ ready with your AssemblyAI API Token?? If not, totally chill, I got you. Here's how to get yours.

You can sign up for a [free AssemblyAI account](https://app.assemblyai.com/login/) in seconds by just entering your email address. After you verify your account from your email address, you're taken right back to your new account where you can see your API token in your dashboard.
![AssemblyAI_key](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/Assembly_AI_API_Key.png)

Take that API Token and add it to your dotenv file in the same way you did for Zoom's API Keys, with it's own unique variable name.

## Fetching a Cloud Recording from Zoom's API in Python
It is time AT LAST to flex on these codes and do our digital Indiana Jones thang.

Plop these fresh import statements into your notebook:

```
import sys
import time
import requests
import authlib
import os
import urllib.request
from dotenv import load_dotenv
from pathlib import Path
from typing import Optional, Dict, Union, Any
from authlib.jose import jwt
from requests import Response
import http.client
import json
```

Now find your directory path to your icy dotenv file, or move the dotenv file to your notebook directory, and plop that path into this code block where it belongs:
```
env_path = Path('.')/'tokens.env'
load_dotenv(dotenv_path=env_path)
```

Next, you'll scoop those white-hot credentials out of the dotenv file by naming them to some notebooke variables. The notebook variables, which you'll be using in subsequent code, are on the left of the **"="**, and the alias variable you used for that API credential in the dotenv file will be the strings on the right of the **"="** assigner.
```
API_KEY= os.getenv("USE_STEPS_ABOVE_TO_GET_YOUR_VERY_OWN_API_KEY")
API_SECRET= os.getenv("USE_STEPS_ABOVE_TO_GET_YOUR_VERY_OWN_API_SECRET")
ASSEMBLY_AI_TOKEN = os.getenv("SIGN_UP_AT_ASSEMBLYAI_TO_GET_YOUR_VERY_OWN_ASSEMBLYAI_API_TOKEN")
```

Now, you'll create a Zoom class that will generate live JWT's that you'll need to unlock your cloud recordings throughout the rest of the code, Again, shoutout to [Billy Harmawan](https://medium.com/swlh/how-i-automate-my-church-organisations-zoom-meeting-attendance-reporting-with-python-419dfe7da58c) for laying this out.
```
class Zoom:
    def __init__(self, api_key: str, api_secret: str):
        self.api_key = api_key
        self.api_secret = api_secret
        self.jwt_token_exp = 518400
        self.jwt_token_algo = "HS256"

    def generate_jwt_token(self) -> bytes:
        iat = int(time.time())

        jwt_payload: Dict[str, Any] = {
            "aud": None,
            "iss": self.api_key,
            "exp": iat + self.jwt_token_exp,
            "iat": iat
        }

        header: Dict[str, str] = {"alg": self.jwt_token_algo}

        jwt_token: bytes = jwt.encode(header, jwt_payload, self.api_secret)

        return jwt_token
```

Then, you'll create an instance of that Zoom class and use it to generate a live JWT.
```
zoom = Zoom(API_KEY, API_SECRET)
jwt_token: bytes = zoom.generate_jwt_token()
jwt_token_str = jwt_token.decode('UTF-8')
print(jwt_token_str)
```

This is where Zoom's code comes in directly from their [API documentation](https://marketplace.zoom.us/docs/api-reference/zoom-api/users/users). The "Send a Test Request" box at the bottom of that page auto-generated this code when I plugged the full JWT string into the "oauth_access_token" feild, sent the request, and toggled "code generation" to Python, but you don't need to do any of that, just run the resulting code below to get your **USER_ID**.
```
conn = http.client.HTTPSConnection("api.zoom.us")

headers = { 'authorization': 'Bearer ' + jwt_token_str}

conn.request("GET", "/v2/users?page_size=30&status=active", headers=headers)

res = conn.getresponse()
data = res.read()
user_dict = json.loads(data.decode("utf-8"))
USER_ID = user_dict['users'][0]['id']

print(USER_ID)
```

If there are multiple users under a shared account you are using, you may have to login to your zoom account and navigate to **"My Account** to clarify exactly which users have the cloud recordings you want to transcribe.

For this next step too, you may have to specify date ranges in the url request, which the autogeneration in the the [Zoom API Documentation](https://marketplace.zoom.us/docs/api-reference/zoom-api/cloud-recording/recordingslist) show you how to do. The default is to only return the last day's worth of meetings, so if the meeting happened before that, you will have to specify per format.

From here, you can find the **MEETING_ID** of the cloud recording you want to access by running the code below, which was also generated directly from the 
```
conn = http.client.HTTPSConnection("api.zoom.us")

conn.request(
    'GET', '/v2/users/' + USER_ID +
    '/recordings?trash_type=meeting_recordings&mc=false&page_size=30',
    headers=headers
    )

res = conn.getresponse()
data = res.read()
meeting_dict = json.loads(data.decode("utf-8"))
MEETING_ID = str(meeting_dict['meetings'][0]['id'])

print(MEETING_ID)
```


And again, to attain the **download_url**, we'll run this code below generated from Zoom's [API documentation](https://marketplace.zoom.us/docs/api-reference/zoom-api/cloud-recording/recordingget)**
```
conn.request(
    "GET", '/v2/meetings/' + MEETING_ID + '/recordings', headers=headers
    )

res = conn.getresponse()
data = res.read()

response_obj = (data.decode("utf-8"))

print(response_obj)
```

After that, you can now grab the **download_url** as your very own!
```
meeting_dict = json.loads(response_obj)
download_url = meeting_dict['recording_files'][0]['download_url']
download_url
```

One last shoutout to [TriBloom](https://github.com/tribloom/Zoom-Meeting-Download/blob/master/zoom_meeting_download.py) and Caleb Spraul for helping me find this next _single line of code_ that makes your download URL mutully intelligible to both the Zoom and AssemblyAI API's...
```
authorized_url = download_url + "?access_token=" + jwt_token_str
```

## Submit the audio file to AssemblyAI for transcription
The code below uses the **authorized_url** variable to feed the Zoom Cloud Recording mp4 directly into AssemblyAI's transcription neural nets.

```
endpoint = 'https://api.assemblyai.com/v2/transcript'

json = {
    'audio_url': authorized_url
}

heads = {
    'authorization': ASSEMBLY_AI_TOKEN,
    'content-type': 'application/json'
}

resp = requests.post(endpoint, json=json, headers=heads)
print(resp.json())
```

A successful response will look like so, displaying the **"status" : "queued"‍**
```
{
    # keep track of the id for later
    "id": "5551722-f677-48a6-9287-39c0aafd9ac1",
    # note that the status is currently "queued"
    "status": "queued",    
    "acoustic_model": "assemblyai_default",
    "audio_duration": null,
    "audio_url": "some/obscenely/long/zoom/api/url/",
    "confidence": null,
    "dual_channel": null,
    "format_text": true,
    "language_model": "assemblyai_default",
    "punctuate": true,
    "text": null,
    "utterances": null,
    "webhook_status_code": null,
    "webhook_url": null,
    "words": null
}
```

## Get that transcription you've been longing for!!
This code below will check the status of the transcription, and concatenates the transcription id **resp.json()['id']** into the url for you, so it will run sequentially with the previous code.

```
status_point = 'https://api.assemblyai.com/v2/transcript/' + resp.json()['id']

status_header = {'authorization':ASSEMBLY_AI_TOKEN} 

status_check = requests.get(status_point, headers=status_header)

print(status_check.json()['status'])
print(status_check.json()['text'])
print(status_check.json())
```

You'll see in the above code you are printing 3 things. The first print statement **print(status_check.json()['status'])** prints out the status of the transcription, which goes from **"queued"** to **"processing"** to **"completed"**. Audio files take around 25% of their audio duration to complete, so a 10 minute audio file would complete within 2.5 minutes. You'll want to run this above code in a loop until the **status** key shows **"completed"**.

Once the **status** is **"completed"**, the second print statement **print(status_check.json()['text'])** will print out the actual transcription text. And the third print statement prints out the entire API response, with a bunch more meta data like the timings for when each word was spoken, the confidence for each word, and more! You can see a complete example of the API's response JSON on the [AssemblyAI Docs here](https://docs.assemblyai.com/overview/getting-started#result).

AND NOW you know how to transcribe Zoom recordings!

![rdj](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/rdj.jpg)

Ok, I lied, one more shout out to AssemblyAI for creating these neural nets so you don't break your fingers transcribing all that noise!

The full sequential code can be found [in this colab notebook](https://colab.research.google.com/drive/1McxVEEvNUazK4ZVCVqRyqBlPVAInNSjV?usp=sharing)! Happy transcribing!
