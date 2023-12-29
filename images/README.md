# Sales AI whisperer

This tool is designed for salespeople. This is a dashboard that generates real-time insights based on a sales meeting. This tool listen in real-time to the meeting and generate insights on how the meeting is going, advices to help close the deal and a suggestion of products to propose. The idea is to help improve the sales closing rate as well as to promote cross-selling opportunities. A sales can be suggested a product that wasn't necessarly in his original scope and redirect the lead to the relevant person or business line.

For this tool, we consider the context of a sales representative in the spanish group Planeta. We have taken a few of their programs and have included them in our tool.

Contact: adrida.github.io

More on how the tool works below.

## How to run it:

Start by running 

`pip install -r requirements.txt`

In the file `config.py`, in the first line, paste your OpenAI API key. Follow the documentation here to get your key https://platform.openai.com/docs/quickstart/account-setup

Open three different terminals. The reason for the three terminals is to dissociate all the componenent in order to have control over their parameters.

### Terminal 1:

This terminal will run the script to record your microphone and it might ask you for an authorization to access the microphone. The recording is transcribed and saved every 10 seconds in the folder `recordings/`. The transcribed text is in `transcriptions/transcript.txt`. This path can be changed in `config.py` 

To run the recording, run this command:

`python script_run_record`

You should see written in the terminal "Recording" if everything worked so far

### Terminal 2:

This terminal will read the transcript every 10 seconds and use GPT4 (config.py to change, only OpenAI models are available) to generate insights on the current conversation and find relevant programs to propose to the lead.

To run the insights generator, run this command:

`python script_generate_status_live.py`

You should see after 10 seconds the GPT output displayed in the terminal

### Terminal 3:

This terminal will simply launch the Gradio dashboard. To use display and update insights on the current recorded discussion press the "Run Whisperer (REAL TIME)" button. This will update the interface with up to date insights.

To launch the app, simply run

`python app.py`

The terminal should show you an localhost IP address, looking probably something like `http://127.0.0.1:7860`

Then open a new tab in your browser and access this address. You know have Sales AI whisperer set up and ready to help you during your meetings. Future updates and improvements include making this whole process more simple and ideally completely handled in the backend of a fully deployed webapp. 

Let me know if you have suggestions or comments at adamrida.ra@gmail.com :)

## How does it work?

This section will breakdown how the whole tool work and what can be changed in terms of parameters.

### Record and transcription

This part is pretty straightforward as it consists of two sub-parts. The first one handles the recording using the package `sounddevice` and the second one will handle the transcription using OpenAI Whisper Speech to text model.

In `record.py` you can change the `duration` in line 7. This duration corresponds to the length of each recording. By default, the recordings will be saved and transcribed every 10 seconds. Make sure to keep it high enough in order to avoid having cutoffs in the transcription due to short chunks. 

### LLM-based insights

In this part, we take all the transcribed meeting and use an LLM to generate different elements that will help us build the dashboard. This query is done every 20 seconds to make sure it generates insights based on the last transcriptions and don't create a delay when the user asks for an update. The update from the interface do not run the LLM query but rather read the last LLM query output.

We generate the following elements:

- A number representing simply a the probability to close the candidate based on the global feeling of how the meeting is going. This will help build the gauge chart.

- A formatted Markdown string containing all the following elements but displayed in a proper and concise way for a sale to read during a meeting in real time:
    'adjective' : # One or two words to describe the feeling of the candidate and the overall feeling of the meeting,
    'candidate_summary' : # A summary of who the candidate is and what he is looking for. It should only contain efficient keywords, no sentences,
    'status_prospect': #The full markdown string describing as bullet points in a compact and concise way what is the status of the potential client and how close he is to being closed. All informations should be relevant for the sales person and help guide him without overwhelm him as we will read this during the sales meeting,
    'arguments': #The full markdown string describing as bullet points in a compact and concise way what arguments should the salesperson give to close the deal. It also contains actionable insights to drive the conversation.

We add instructions in the prompt regarding the cognitive workload of the sales during the meeting. The idea is to keep it balanced to not overcharge the sales rep with too much information and keep him focused on the conversation. The goal is to keep the dashboard as a kind of head-up display (HUD) that will assist him during the call.

We use GPT-4 by default for this.

Again, in `config.py` some parameters can be adapted to your usecase. The first one is the OpenAI model used. You can replace the `LLM_MODEL` variable with another gpt model from the OpenAI API (check out the official documentation for an exhaustive list but it needs to be a chat model)

Another parameter that can be changed is the `PROMPT_INSTRUCTIONS` variable. It corresponds to the prompt itself and can be tuned and adapted to what you need to generate. Keep in mind that changes will necessarly trigger changes in the Gradio App in order to make sure that the data is correctly parsed.

Last parameter that can be changed is the frequency of the LLM update. This can be changed in line 54 of `script_generate_status_live.py` :
`schedule.every(20).seconds.do(execute_generate_gpt)`

You can change 20 to the frequency you like. A longer duration will have as effect that insights displayed in real time might be a bit old. A shorter duration can lead to very up to date insights but can have delays displayed in the dashboard as the LLM query will almost be real time. Also a quick frequency can increase very highly the API costs behind.

### Gradio App

The gradio app is composed of different components. We bind the update button to the `get_insights()` function that will read the last LLM output and parse it into different components and actions.

It will generate a gauge chart with the probability of closing the deal and general insights as described above. It will also perform a semantic search on the database of available programs in order to suggest to most suited program to the lead. There is also the possibility to perform this seamantic search manualy. The source of programs come from the group Planeta and is only a small sample.


### Contact and contribution

This tool has been made as an exploratory projects to try out what could be done using the power of LLMs. If you have any questions or remarks, please feel free to get in touch at adamrida.ra@gmail.com or visit my website: adrida.github.io