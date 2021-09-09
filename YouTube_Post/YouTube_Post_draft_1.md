#Transcribing YouTube Videos Using AssemblyAI in Python

In this post, we’re going to show you how to transcribe YouTube videos, by connecting just the video url to AssemblyAI’s automatic speech recognition API in Python. More specifically, we’ll walk you through:

- Using python packages to convert a YouTube URL into a transcribable audio file
- Signing up for an AssemblyAI API token
- Submitting the audio file to AssemblyAI for transcription
- Feasting your eyes on that transcription you’ve long awaited!

I'm going to demonstrate this transcription process on a seminal video from my past, which was  influential in my decision to go into Machine Learning Engineering. The video, about the economics of automation, is called [Humans Need Not Apply](https://www.youtube.com/watch?v=7Pq-S557XQU&t=13s) by one of my favorite online intellectuals, [CGP Grey](https://www.youtube.com/user/CGPGrey).

Buckle up nerds! This roller coaster's fast, blink and you'll miss it!

## Strip the Audio from the YouTube video: Python Packages For the Win

Crack open your code editor, and feed into it the following magic commands:

```
!pip install pafy
!apt install ffmpeg
!sudo pip install --upgrade youtube_dl
```

*Pafy*, in tandem with *youtube_dl* iterate through chunks of the YouTube video feed to convert it into whichever video format will optimize the quality of the stripped audio data, usually a .webm file. And the cherry to top that sundae is the *ffmpeg* package, which will strip the audio of the video file, outputting a transcription-ready audio file.

In this sample code, we will convert it to good ol' reliable mp3 to feed it into AssemblyAI for transcription, [though AssemblyAI can handle many ubiquitous audio and video formats](https://docs.assemblyai.com/overview/supported-file-formats).

Use the following code, courtesy of [Priya Ganjikunta](https://knowpythonwithpriya.wordpress.com/2017/09/12/youtube-url-to-mp3-converter/), to extract an audio file from your YouTube URL.
```
import pafy
import subprocess
import os
import sys

URL="https://www.youtube.com/watch?v=7Pq-S557XQU&t=13s"

video = pafy.new(URL)
title = video.title.replace(" ", "_")

bestaudio = video.getbestaudio()
BESTFILE = os.getcwd() + "/" + str(title) + "." + str(bestaudio.extension)
MP3FILE = os.getcwd() + "/" + str(title) + ".mp3"
bestaudio.download(BESTFILE)

command = "ffmpeg -i "+str(BESTFILE)+" -vn -ab 128k -ar 44100 -y "+str(MP3FILE)
print("Command to convert audio file to .mp3 format: ", command)
subprocess.call(command, shell=True)
os.remove(BESTFILE)
```

That should drop a fresh mp3 into your working directory, which you can play to check that the packages did their job.

Now, as the roller coaster click-click-clicks closer to its apex, we must register with AssemblyAI's API to bring us over the hump!

## Sign Up for an AssemblyAI API Token

You can sign up for a [free AssemblyAI](https://app.assemblyai.com/login/) account in seconds by just entering your email address. After you verify your account from your email address, you're taken right back to your new account where you can see your API token in your dashboard.

![AssemblyAI_key](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/Zoom_API_Post/screencaps/Assembly_AI_API_Key.png)

Take that API Token and add it to a **tokens.env** file in your working directory, with it's own unique variable name. You can create this file in a simple text editor or code editor. This is what the text of your **tokens.env** file should look like:

```
ASSEMBLYAI_TOKEN = COPY_PASTA_THAT_API_KEY_HERE_ONCE_YOU_SIGN_UP_AT_ASSEMBLYAI
```

And now... the roller coaster *drops*. WOOOH! Let's transcribe with AssemblyAI's API!

## Submit the MP3 to AssemblyAI for Transcription

This next code block allows you to use the variable in your **tokens.env** file to feed your precious, covert AssemblyAI API key into your transcription code anonymously. If others see your transcription code, they will not be able to see your API key without the **tokens.env** file.

```
!pip install python-dotenv

from dotenv import load_dotenv
from pathlib import Path
import os

env_path = Path('.')/'tokens.env'
load_dotenv(dotenv_path=env_path)
ASSEMBLY_AI_TOKEN = os.getenv("ASSEMBLYAI_TOKEN")
```

Now, let's upload that mp3 file to AssemblyAI.
```
import sys
import time
import requests

filename = MP3FILE

def read_file(filename, chunk_size=5242880):
  with open(filename, 'rb') as _file:
    while True:
      data = _file.read(chunk_size)
      if not data:
        break
      yield data

headers = {'authorization': ASSEMBLY_AI_TOKEN}

response = requests.post(
    'https://api.assemblyai.com/v2/upload',
    headers=headers, data=read_file(filename)
)

print(response.json())
```

A successful upload will yield a JSON response similar this:
```
{"upload_url": "https://cdn.assemblyai.com/upload/ccbbbfaf-f319-4455-9556-272d48faaf7f"}
```

The code below uses the **upload_url** key value from the JSON response to queue the audio file for transcription.
```
endpoint = 'https://api.assemblyai.com/v2/transcript'

json = {
    # 'audio_url': response.json()['upload_url']
    'audio_url': response.json()['upload_url']
}

heads = {
    'authorization': ASSEMBLY_AI_TOKEN,
    'content-type': 'application/json'
}

resp = requests.post(endpoint, json=json, headers=heads)
print(resp.json())
```

A successful response will look like so, displaying the **"status" : "queued"**
```
{
    # keep track of the id for later
    "id": "5551722-f677-48a6-9287-39c0aafd9ac1",
    # note that the status is currently "queued"
    "status": "queued",    
    "acoustic_model": "assemblyai_default",
    "audio_duration": null,
    "audio_url": "https://s3-us-west-2.amazonaws.com/blog.assemblyai.com/audio/8-7-2018-post/7510.mp3",
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

This code below will check the status of the transcription...

```
status_point = 'https://api.assemblyai.com/v2/transcript/' + resp.json()['id']

status_header = {'authorization':ASSEMBLY_AI_TOKEN} 

status_check = requests.get(status_point, headers=status_header)

print(status_check.json())
# print(status_check.json()['text'])
```

A successful response will display **"status" : "processing"**.

Audio files take around 25% of their audio duration to complete, so a 10 minute audio file would complete within 2.5 minutes. You'll want to run the following code, which will loop until the **"status"** key shows **"completed"**.

```
while status_check.json()['status'] == 'queued' or status_check.json()['status'] == 'processing':
  status_check = requests.get(status_point, headers=status_header)
  continue

print(status_check.json()['status'])
print('\n', status_check.json()['text'])
```

And there it is! CGP Grey's wisdom transcribed, by the very technology which he ponders in "*Humans Need Not Apply*", faster than any human could ever transcribe.

```
completed

"Every human used to have to Hunt or gather to survive. But humans are smartly,
lazy. So we made tools to make our work easier, from sticks to plows to
tractors. We've gone from everyone needing to make food to modern agriculture
with almost no one needing to make food. And yet we still have abundance. Of
course, it's not just farming, it's everything..."
```

Take a gander with a favorite YouTube video of your own, and happy transcribing!

![Ex_Machina_Meme](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/YouTube_Post/ex_machina_meme.JPG)