# How to transcribe an audio file with Python and AssemblyAI (freemium! 8D )

```
This introduction paragraph is special. Why is it special? Well, if you're 10
moves ahead of me like a chess Grandmaster, then you've probably already guessed
it. This introduction was not hand Typed. It was spoken and then transcribed
with Python using the Assembly AI API.
```

Seriously. Don't believe me? [Here's the audio recording](https://github.com/ddodds42/sandbox/blob/master/AssemblyAI_howto_intro2.MP3?raw=true) of my sweet, sultry voice creating that paragraph. I spoke it off the cuff, and it only took me 2 takes to not stumble over my own words that I imagined. XD

![Inception](https://github.com/ddodds42/sandbox/raw/master/inception_transcription.jpg)

I would have spoken this entire post by voice, but because of my limited human memory, I would have to script it in writing, and that defeats the proof of concept, now doesn't it?

From there, I fed the mp3 to [my pet python](https://colab.research.google.com/drive/12ZmHK6cplmhDOh9gWXgonWG_sw32VSpF?usp=sharing), and with the help of [AssemblyAI](https://www.assemblyai.com/), my pet python delivered unto the world that beautifully transcribed paragraph.

Here's how you can too.

## 1 - Smash that Sign Up button

I was able to sign up to [AssemblyAI](https://www.assemblyai.com/) in one bounce between their page and my email. It's as simple as verifying your account with a click, and it takes you right back to your new account, API token, and a [Quickstart Guide](https://docs.assemblyai.com/) in AssemblyAI's docs.

## 2 - Find an audio file to try it on

AssemblyAI transcription supports loads of [file types](https://docs.assemblyai.com/overview/supported-file-formats), and gives you 5 hours of audio footage transcription per month absolutely free, with no billing setup or commitment.

So find that perfect mic check audio file you want to try it out on, you'll have plenty of chances to test it with our API.

## 3 - Upload your File for transcription

It's not absolutely necessary if you have a url for the audiofile already housed elsewhere... but if you don't want to use space on your servers for the test, you can upload it to AssemblyAI with this code:

```
import sys
import time
import requests

filename = '/content/AssemblyAI_howto_intro2.MP3'

def read_file(filename, chunk_size=5242880):
  with open(filename, 'rb') as _file:
    while True:
      data = _file.read(chunk_size)
      if not data:
        break
      yield data

headers = {'authorization': 'GET_YOUR_OWN_API_KEY_YOU_CANT_HAVE_MINE'}

response = requests.post(
    'https://api.assemblyai.com/v2/upload',
    headers=headers, data=read_file(filename)
)

print(response.json())
```

A successful upload will yeild a JSON response similar this:
```
{"upload_url": "https://cdn.assemblyai.com/upload/ccbbbfaf-f319-4455-9556-272d48faaf7f"}
```

## 4 - Submit the audio file for transcription

The code below uses the ```'upload_url'``` key value from the JSON response in the previous code, so it will run sequentially with that. If you already have a url for your audio file, replace it with ```response.json()['upload_url']```

```
endpoint = 'https://api.assemblyai.com/v2/transcript'

json = {
    'audio_url': response.json()['upload_url']
}

heads = {
    'authorization': 'GET_YOUR_OWN_API_KEY_YOU_CANT_HAVE_MINE',
    'content-type': 'application/json'
}

resp = requests.post(endpoint, json=json, headers=heads)
print(resp.json())
```

A successful response will look like so, displaying the ```"status" : "queued"```

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

## 5 - Get that transcription you've been longing for!!

This code below will check the status of the transcription, and concatenates the transcription id ```resp.json()['id']``` into the url for you, so it will once again run sequentially with the previous code.

```
status_point = 'https://api.assemblyai.com/v2/transcript/' + resp.json()['id']

status_header = {'authorization':'GET_YOUR_OWN_API_KEY_YOU_CANT_HAVE_MINE'} 

status_check = requests.get(status_point, headers=status_header)

print(status_check.json())
# print(status_check.json()['text'])
```

A successful response, displaying the ```"status" : "completed"```, as well as the text of your transcription, which you can distill out with the code:

```
print(status_check.json()['text'])
```

My mom did manual transcription work in the 90's, and she hated it. REJOICE at this beautiful API that AssemblyAI has created for you, and all the headaches it can save you.

![I made this for you](https://github.com/ddodds42/sandbox/raw/master/I%20made%20this%20for%20you.jpg)

Links to AssemblyAI's doc pages on this can be found [HERE](https://docs.assemblyai.com/guides/uploading-audio-files-for-transcription), [HERE](https://docs.assemblyai.com/guides/transcribing-an-audio-file-recording), and [HERE](https://docs.assemblyai.com/overview/getting-started); and the full sequential code with my pet python [HERE](https://colab.research.google.com/drive/12ZmHK6cplmhDOh9gWXgonWG_sw32VSpF?usp=sharing).

### Happy transcribing!
David