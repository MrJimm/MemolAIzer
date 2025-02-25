Tool to automatically convert your voice memos to text and analyze it with ChatGPT.
It consists of N8N workflows and uses Google Drive to sync and store all processed data.

![image_2024-05-08_00-01-19](https://github.com/MrJimm/MemoAIzer/assets/5428408/e590fd4b-351e-457b-8904-83868acf421d)

# TL;DR
1. Install n8n ([docker](https://docs.n8n.io/hosting/installation/docker/), see sections 0.1-0.2 for details).
2. Import voice_memo_transcribe.json workflow from json file.
3. Set up credentials for Google Drive and OpenAI nodes (see details at "Docs" tab of a node).
4. Create a root folder on your Google Drive, where all data will be stored
5. Link this root folder in "Root folder" node (dropdown near "from list" field)

      <img src="https://github.com/MrJimm/MemoAIzer/assets/5428408/8ff970a9-40fd-46af-9b2c-ec6d52f5e5d0" width="200">


7. Run the voice_memo_transcribe flow - it will automatically create all the folders hierarchy inside the root folder on your Google Drive.
8. Now you can place some voice data into [root]/voice/raw/general and wait for transcription results in [root]/ 
9. To engage text processing, import text_memo_analysis.json flow, select your root folder in the "Root folder" node (see section 2 for details) 
10. Activate text analysis flow and wait for results to appear in folders in [root]/text/processed/base

NOTE: Since flows are triggered by a timer hourly, you can either run each flow manually with the 'Test workflow' button in the N8N UI to get immediate results, or adjust the timer trigger node "Schedule trigger" and restart the flow.

  <img src="https://github.com/MrJimm/MemoAIzer/assets/5428408/7433314f-af39-4c43-98f3-4d5add0ba8b2" width="600">


# Detailed instruction

## Part 0: Setup prerequisites
### 0.1 Install N8N
Install N8N from the official website, or get official docker image from hub (seem easier way, see instruction https://docs.n8n.io/hosting/installation/docker/).

Set N8N_PAYLOAD_SIZE_MAX env variable to some high value (123456 will be enough), so you can move large files in your N8N flow over the internet.

If you use docker image - set this env variable for the container before you run it. Also don't forget to forward the port (default is 5678).

#### 0.1.1 Instruction for Windows
Here is step-by-step instruction for Windows installation with docker
1. Install Docker for Windows from https://docs.docker.com/desktop/install/windows-install/
2. Run Docker Desktop app
3. Go to the searchbar at the top, enter "n8n"
   
      <img src="https://github.com/MrJimm/MemoAIzer/assets/5428408/a9aeb615-1192-4806-a98b-de6956552e1b" width="400">
      
5. Find n8n image from n8nio (the one with the most downloads), press "Pull"
6. After the image is downloaded, press the "Run" button. You will see a manu
7. Expand "Optional settings" and set value for N8N_PAYLOAD_SIZE_MAX as 1234. This variable sets max payload size in megabytes, helps to avoid an error when you try upload large audio files
      (details on this env variable here https://docs.n8n.io/hosting/configuration/environment-variables/endpoints/)
   
      <img src="https://github.com/MrJimm/MemoAIzer/assets/5428408/6bdb5d88-8978-4368-8fda-c031fab5bd10" width="400">
      
9. Optionally you can set container name in the first field
10. Hit "Run". After a few seconds N8N UI will be available in browser at http://localhost:5678/

### 0.2 Start N8N and import workflow
Start N8N service/container and open web interface (the default is http://localhost:5678).

At first start it will require you to pass the simple sign up procedure.

Then create a new workflow, press three dots button at right upper corner, select "Import from File" and select file of Voice memo transcribe flow (voice_memo_transcribe.json).
Save the flow (also you can change its name, if you click on its name "My workflow" in the upper panel).

### 0.3 Create OpenAI and Google Drive credentials
In the imported Workflow open any Google Drive node (i.e. "New in general folder"), click on "Docs" tab, and follow instructions to create credentials, that you can use to connect to your Google Drive account.

![изображение](https://github.com/MrJimm/MemolAIzer/assets/5428408/3944f783-c4ae-4e89-9564-38a8bca25b0e)

TIP: It can be a tricky process. Note, that you should create Credentials and Publish app on "OAuth consent screen"
![изображение](https://github.com/user-attachments/assets/019eff5a-f121-4b68-97d0-6ab60d3c0f53)

After you created Drive credentials, open each Google Drive node and choose it. Do the same for Whisper transcription node to create OpenAI API credentials.

### 0.4 Google Drive folders hierarchy ()
After you run the voice transcription flow for the first time, it will create the 'voice' folder hierarchy within the selected root folder. It will also create a 'text/raw' folder, where all transcriptions will be stored.

```
|── root
  |── voice
      |── raw
            |── general
      |── processed
            |── general_processed
      |── error
            |── general_error
  |── text
      |── raw
      |── processed
            |── base
```

#### Voice
The "voice" folder is where all my input voice memo files are stored. Subfolders in the "raw" folder (*device* subfolders) are where you put all your original voice memo files.
**Initially, there's only the "general" subfolder – start by placing your voice memo files here.** If you need another *device* subfolder - just add it to the "raw" folder.

Subfolders in the "processed" folder are where the flow will copy voice memo files after successful transcription. This allows the flow to detect new memos. I used the "copy" operation instead of "move" here because the tool I use to automatically upload voice memos from my smartwatch checks the files list in the original directory. Also, it's safer to use non-destructive operations. :)

Subfolders in the "error" folder are where voice memos will be copied if the Whisper node returns an error during transcription (this usually happens if a voice memo file is too large or your quota has expired). This error won't stop the flow. If you want these memos to be reprocessed, just remove the files from the corresponding "error" folder.

#### Text
"text" folders are for text memos. All successful transcriptions are automatically placed in "raw" folder. You also can place there your custom .txt file with text memo for further processing.
All text analysis results are stored in "processed" folder.


## Part 1: Transcribe voice memo to text

### 1.1 Activate N8N flow
Finally, activate your flow. You can then go to the "Executions" or "All executions" tab to monitor automation activities.
The transcription flow is triggered by the timer node. The initial period is set to one hour, which you can adjust in the "Schedule Trigger" node (the first one in the flow).

Memoaizer works with voice memos stored in Google Drive, in *devices* subfolders within root/voice/raw. To set up automatic upload from your phone/watch see section "How to upload to Drive" below. Also see troubleshooting section if you see no execution is fired while you have new files in *device* folders. You can see executions in the designated tab in N8N interface.

## Part 2: Analyze text transcriptions with ChatGPT API 
Import text memo analysis flow from a text_memo_analysis.json. This flow will analyze your text transcriptions with ChatGPT API. The results of base processing for each memo, that you can find in subfolders in [root]/text/processed/base are: 

1. XML with two titles and keywords, 
2. Original text, formatted and cleaned
3. Summary of the note in the original language
4. Summary of the note in English
5. Copy of the original text of the transcription

### 2.1 Load text analysis flow from the file
1. Load text memo analysis flow from the text_memo_analysis.json.
2. For evey Google Drive and OpenAI API block set-up credentials as you did above.

![изображение](https://github.com/MrJimm/MemoAIzer/assets/5428408/928452cf-32fc-4f77-9756-7071024ec45c)

(red exclamation mark signs near Google Drive and OpenAI blocks highlights where you should set up credentials)

3. Re-select your Google Drive root foolder in the "Root folder" node

![изображение](https://github.com/MrJimm/MemoAIzer/assets/5428408/bfc8f2d7-77f3-4e29-9363-fef6131f93c6)

7. Launch the flow (hit "Active" button)

### 2.2 Managing text processing prompts
You can find prompts for text processing inside OpenAI blocks (i.e. "Get keywords"). It is plain text, and I suggest to read it to understand how exactly it asks ChatGPT to process your text memos. You can change this prompts and add new branches with custom processing prompts. Though I plan to introduce a more convenient way to add custom text processing flows in further updates.

![изображение](https://github.com/MrJimm/MemoAIzer/assets/5428408/a94b9ea8-c2aa-4ed8-8fc5-8e4ddd998884)


## How to upload to Drive 
### Manually
It is what it is:) Just place voice memo file it in the specific (i.e. "general") folder.
Also you can place text memos (in .txt format) straight to the text/raw folder, so they will be processed as regular transcriptions. But be aware of the size of context window of GPT model you use, so your custom text will fit it.

### Android
I use pro version of FolderSync app to automatially sync recordings files from some particular folder on a phone to "watch" folder on my Google Drive. This app allows to speify particular folder, where all recordings only from watch are stored, and "to" folder on Google Drive.  

### Watch: example for Galaxy Watch 3 and Samsung S23 Ultra:
For Galaxy Watch 3 on Android in the Wearable app go to Apps -> Voice Recorder -> Settings (gear icon) and toggle "Auto copy to phone". 
Then all recordings will sync in the folder /storage/emulated/0/Recordings/Sounds/Gear (at least, on my setup. Check similar path if memos will not appear there). Now you can use this path on your Phone to your watch recordings in sync app (FolderSync Pro in my case) to automatically sync memos to Drive ("watch" folder in my case).

## Troubleshooting
1. If you have unprocessed memo files in your upload folder, but N8N don't fire execution - stop it and try to run with a "Test workflow" button in the UI. This way you can see the instant flow execution in realtime. Also try to adjust the "Schedule Trigger" timer period value.
2. Though voice transcription flow shuffles new unprocessed files, sometimes flow can stuck on "bad" file. For this I occasionaly check execution status and if I notice such problematic file, I manually move it to specific "*_error" folders, that I created in my "voice" folder.
3. If your text memo processing flow does nothing after you started it, try to delete and then add manually any text file in "processed" folder. This should pull the right trigger in the flow. 
