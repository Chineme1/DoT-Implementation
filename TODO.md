This is a list of things that you can do to help make running DoT easier.

Below are places you can use to run the code in the order I recommend.

1. Go to Code(main) → Codespaces → Create codespace
2. Run locally
3. Lighting AI studios
4. Kaggle Notebooks

In Github for the Codespaces ensure that you pick the machine with 4 cores.
After creating the codespace
Go back to the repo
Click on Code
Click on the 3 buttons right next to the active codespace
Click on the Change machine type
Click on the 4-cores

In any case, I think we can focus on the math section of DoT(MATH_Trys folder) to get and understand
Make and activate a virtual environment:
python -m venv venv
source venv/bin/activate

Install dependencies:
pip install tqdm
pip install ipydagred3
pip install networkx
pip install openai
pip install numpy

Install ollama:
curl -fsSL https://ollama.com/install.sh | sh
ollama serve &
ollama pull llama3.1:8b

Get an OpenAI key(with an OpenAI account) and put it into line 149 in the utils.py file, where it says api_key.

Run this "python MATH_dotrun_step1.py/MATH_dotrun_step2.py" file.
I added SLMC for you all to search to see the changes that are and the hyperparameters that have to be changed.

So you type in SLMC(CMD/CTrl + F; Type in "SLMC") to see all the changes to be made.

I made a couple of edits in the math folder and the file.
You can find it by typing (CMD/CTrl + F; Type in "SLMC")

A couple of prompts in functions to be modified are in MATH_utils.py and these changes can be applied to other files as well:

askLLM - has the different models D.o.T uses
decompose_sql - Has prompt to decompose problems
allo_model - works on the allocation of models to specific subtasks


Changing and creating the prompts for the LLM to use where it can automatically switch between tasks.

Not sure why they have to give the Mohammed example exactly, why can't they just use something that's not as specific
Like having a harness that can read a question and decide what category it is and what tools to use based on the question.


Unfortunately, some of the other datasets require a Baidu account and to set up a Baidu, one needs a Chinese number which I unfortunately don't have. There are a couple of system path assignments that have to be removed 

I think we might have to update the MATH tests to what is being used to test 2026 LLMs.
Also utilizing harnesses might be a good idea for some of these problems to generalize them in order for them not to have specific utils page and the agent can pick a particular harness based on the problem
The harness would also decide what should solve the problem either the SLM or LLM model.
And piece it together. What probably can be changed  might be the prompt to emphasize what the user wants to focus on. First let's get this running and working well on your individual computer before working on indivual fixes and  improvements.
