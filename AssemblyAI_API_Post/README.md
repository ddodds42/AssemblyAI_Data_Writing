# [How to transcribe an audio file with Python and AssemblyAI](https://www.assemblyai.com/blog/how-to-transcribe-an-audio-file-with-python-and-assemblyai)
Link above is the post on AssemblyAI's blog.
Below is the same article.

#How to transcribe an audio file with Python and AssemblyAI
### APRIL 30, 2021

```
This introduction paragraph is special. Why is it special? Well, if you're 10
moves ahead of me like a chess Grandmaster, then you've probably already guessed
it. This introduction was not hand Typed. It was spoken and then transcribed
with Python using the Assembly AI API.
```

Seriously. Don't believe me? Below is the audio recording of my sweet, sultry voice creating that paragraph. I spoke it off the cuff, and it only took me 2 takes to not stumble over my own words that I imagined 😅.

![inception](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/AssemblyAI_API_Post/inception_transcription.jpg)

I would have spoken this entire post by voice, but because of my limited human memory, I would have to script it in writing, and that defeats the proof of concept, now doesn't it? Here's how you can transcribe your audio files with Python too.

## Sign up for an AssemblyAI API Token
You can sign up for a [free AssemblyAI account in](https://app.assemblyai.com/login/) seconds by just entering your email address. After you verify your account from your email address, you're taken right back to your new account where you can see your API token in your dashboard.

![AssemblyAI_token](https://github.com/ddodds42/AssemblyAI_Data_Writing/raw/master/AssemblyAI_API_Post/AssemblyAI_token.png)

## Find an audio file to try it on
AssemblyAI transcription supports loads of [file types](https://docs.assemblyai.com/overview/supported-file-formats), and gives you 5 hours of audio transcription per month absolutely free, with no billing setup or commitment. So find that perfect mic check audio file you want to try it out on, you'll have plenty of chances to test it with our API.

## Upload your file for transcription
If you already have a publicly accessible URL for your audio file already hosted somewhere, you can skip this step... but if not, the first step is to upload your audio file to AssemblyAI with this code:

```
import sys
import time
import requests

filename = '/path/to/your/audio_file.mp3'

def read_file(filename, chunk_size=5242880):
  with open(filename, 'rb') as _file:
    while True:
      data = _file.read(chunk_size)
      if not data:
        break
      yield data

headers = {'authorization': 'YOUR_OWN_API_KEY'}

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

## Submit the audio file for transcription
The code below uses the **upload_url** key value from the JSON response in the previous code, so it will run sequentially with that. If you already have a url for your audio file, replace it with **response.json()['upload_url']**.

```
endpoint = 'https://api.assemblyai.com/v2/transcript'

json = {
    'audio_url': response.json()['upload_url']
}

heads = {
    'authorization': 'YOUR_OWN_API_KEY',
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
This code below will check the status of the transcription, and concatenates the transcription id **resp.json()['id']** into the url for you, so it will once again run sequentially with the previous code.

```
status_point = 'https://api.assemblyai.com/v2/transcript/' + resp.json()['id']

status_header = {'authorization':'YOUR_OWN_API_KEY'} 

status_check = requests.get(status_point, headers=status_header)

print(status_check.json()['status'])
print(status_check.json()['text'])
print(status_check.json())
```

You'll see in the above code we are printing 3 things. The first print statement **print(status_check.json()['status'])** prints out the status of the transcription, which goes from **"queued"** to **"processing"** to **"completed"**. Audio files take around 25% of their audio duration to complete, so a 10 minute audio file would complete within 2.5 minutes. You'll want to run this above code in a loop until the **status** key shows **"completed"**.

Once the **status** is **"completed"**, the second print statement **print(status_check.json()['text'])** will print out the actual transcription text. And the third print statement prints out the entire API response, with a bunch more meta data like the timings for when each word was spoken, the confidence for each word, and a bunch of other data! You can see a complete example of the API's response JSON on the [AssemblyAI Docs here](https://docs.assemblyai.com/overview/getting-started#result).

My mom did manual transcription work in the 90's, and she hated it. REJOICE at this beautiful API that AssemblyAI has created for you, and all the headaches it can save you.

![for_you]()
