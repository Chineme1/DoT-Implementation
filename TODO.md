Getting Started with DoT (Division-of-Thoughts)

A step-by-step guide for running, understanding, and improving DoT. Work through the phases in order. Each phase has a clear goal and a "done when" check, so you always know whether you can move on.

1. The Big Picture

What DoT does. Given a problem, DoT (1) decomposes it into sub-tasks, (2) builds a dependency graph between them, (3) assigns each sub-task to a small local model (SLM) or a large cloud model (LLM) based on difficulty, then (4) solves the sub-tasks in graph order and combines the answers. The goal is to keep accuracy close to an all-LLM approach while cutting cost and latency.

Your goals, in order:

Run it. Get MATH_dotrun_step1.py running end to end on your own setup.
Understand it. Know what each stage does and which function is responsible for it.
Measure it. Get a baseline accuracy number on the MATH questions.
Improve it. Pick one focused change (prompts, allocation, benchmarks, generalization) and show it helps.

Scope. Focus on the MATH section (MATH_Trys/) only. Other benchmarks are optional (see Datasets).

2. Roadmap
Phase	Goal	Done when
0. Read	Understand the paper and the 4 pipeline stages	You can explain decompose, allocate, dependency graph, and solve in your own words
1. Set up	Working environment with both models reachable	ollama list shows a model and your OpenAI key is set
2. First run	One question runs start to finish	The log ends with Correct_Q or False_Q of 1, and Error_Q: 0
3. Fix known issues	Remove the bugs listed in section 5	Your run does not silently skip steps
4. Baseline	Accuracy on a batch of MATH questions	You have a number and a saved log
5. Improve	One focused experiment	You can compare it against your baseline
3. Phase 0: Read Before You Run
Read the abstract and the method section of the DoT paper. Draw the pipeline by hand.
Read MATH_Trys/MATH_dotrun_step1.py top to bottom once. Don't try to understand every line, just follow the order of function calls.
Skim MATH_Trys/MATH_utils.py and utils.py to see where each stage lives (see Code Map).
4. Phase 1: Setup
4.1 Where to run it (in the order I recommend)
GitHub Codespaces (easiest, nothing to install locally)
Run locally
Lightning AI Studios
Kaggle Notebooks
4.2 Codespaces setup
Go to the repo, then Code (main) → Codespaces → Create codespace.
Switch to the 4-core machine: go back to the repo, click Code, click the ⋯ next to your active codespace, then Change machine type → 4-core.
Codespaces have no GPU. Big models will be slow (see the model note in 4.5).
4.3 Python environment
bash
python -m venv venv
source venv/bin/activate
pip install tqdm ipydagred3 networkx openai numpy
4.4 OpenAI API key (do this safely)

Get a key from your OpenAI account. Never paste the key into a file that you commit. GitHub will block the push, and the key must then be revoked.

Set it as an environment variable instead:

bash
export OPENAI_API_KEY="your-key-here"

(In Codespaces, you can store it under Settings → Codespaces → Secrets so it loads automatically.)

Then make setOpenAi() in utils.py read it:

python
def setOpenAi(keyid=0):
    api_key = os.environ.get("OPENAI_API_KEY")
    if not api_key:
        raise RuntimeError("OPENAI_API_KEY is not set.")
    client = OpenAI(api_key=api_key)
    addtoken(-1)
    return client

Also make sure .gitignore contains __pycache__/, *.pyc, and .env.

4.5 Local model with Ollama
bash
curl -fsSL https://ollama.com/install.sh | sh
ollama serve > ollama.log 2>&1 &
ollama pull llama3.1:8b
ollama list

Every time you open a new Codespace session, make sure Ollama is running again. If you see Connection refused on localhost:11434, it is not running.

Model size note. On a CPU-only 4-core machine, an 8B model can be very slow or run out of memory. If so, use a smaller model such as llama3.2:3b for development.

Model names must match. The allocator outputs names like llama3-8b, but Ollama only knows the tags you pulled. Add a map in utils.py (llama branch of askLLM) so the two line up:

python
LOCAL_MODEL_MAP = {
    'llama3-8b': 'llama3.1:8b',
    'llama3-70b': 'llama3.1:8b',   # no local 70B; use what you pulled
    'llama3.1:8b': 'llama3.1:8b',
}
response = client.chat.completions.create(model=LOCAL_MODEL_MAP[model], ...)
5. Phase 2 and 3: First Run and Known Issues
5.1 First run
bash
cd MATH_Trys
python MATH_dotrun_step1.py

N in MATH_dotrun_step1.py is how many questions to run. Keep it at 1 until a single question completes without errors.

Make debugging easier first. In MATH_dotrun_step1.py, change the bare except: so errors are not hidden:

python
except Exception:
    import traceback; traceback.print_exc()
    attempts += 1
    logger.info(f"error: {attempts};  taskid: {question_id}")
5.2 Known issues in the current code

Fix these before trusting any accuracy numbers.

#	File / function	Problem	Fix
1	MATH_utils.py / AllocateModel	eval() on the model's reply crashes on code fences or a wrong tuple count	Parse with ast.literal_eval plus a regex fallback, and pad or trim to the number of steps
2	MATH_dotrun_step1.py	for i in range(MAXHeight) skips the deepest graph level, so the final sub-task is never solved	Use range(MAXHeight + 1)
3	MATH_dotrun_step1.py	Sub-tasks with no dependency edges are dropped	After computing depths: node_depths.setdefault(f"Step {k}", 0) for every step
4	utils.py / askLLM	The GPT branch hardcodes model='gpt-4o', so the allocation is ignored for GPT models	Use model=model
5	utils.py / askLLM_withprob	messages = 'gpt-4o' (should be messages), and 'gpt-4o' is missing from the model list	Fix both. Needed for step 2 style runs
6	MATH_dotrun_step1.py	step1Res is never filled, so the output JSON is empty	Store steps, allocation, and edges per question
7	utils.py / setOpenAi	Always raises if the key line is left empty	Read the key from the environment (see 4.4)

Done when: one question completes and the log shows Correct_Q: 1 or False_Q: 1 with Error_Q: 0.

6. Phase 4: Baseline
Set N = 10 first, then N = 100. Runs call paid APIs, so check the token and cost lines at the end of the log.
Record: accuracy, total tokens, total cost, runtime, and which models the allocator chose.
Temperature is 1, so results vary run to run. Run the same setting a few times and report the range.
Save your logs. Every later change is compared against this baseline.

The MATH test set is old. For serious claims, we probably need to swap in the benchmarks used to evaluate 2026 LLMs.

7. Phase 5: Improve It

First get the baseline working on your own machine. Then pick one idea below and test it.

7.1 Where the interesting work is
Allocation (AllocateModel). This is where most of the time should go. Right now the prompt only considers difficulty, capability, and cost. The research opportunity is to also factor in privacy, latency, and other system factors when deciding SLM versus LLM.
Decomposition (decompose_sql). The prompt uses one fixed few-shot example (the Mohamed/Kody problem). Does that bias the decomposition? Try different examples, no example, or examples matched to the problem type.
Generalization. Instead of a separate utils file per benchmark, build a harness that reads a question, decides its category, picks tools, and decides whether the SLM or LLM should solve it, then pieces the answer together.
Evaluation. Update the benchmark to something current.
7.2 How to run an experiment
Change one thing.
Run the same questions as your baseline.
Compare accuracy, cost, and latency.
Write down what you expected and what happened, including when it did not help.
8. Reference
Code map
What	Where
Model calls (GPT and Llama routing)	utils.py: askLLM
Decomposition prompt	MATH_Trys/MATH_utils.py: decompose_sql
SLM/LLM allocation prompt	MATH_Trys/MATH_utils.py: AllocateModel
Dependency graph	construct_dependencies_without_traversal, create_dag_from_string, calculate_node_depths
Main loop and judging	MATH_Trys/MATH_dotrun_step1.py

I marked my edits and the settings worth tuning with the tag SLMC. Search for SLMC (Cmd/Ctrl + F) to find them all.

Common errors
Error	Meaning	Fix
Connection refused on localhost:11434	Ollama is not running	ollama serve &
404 / model not found from Ollama	Model name does not match a pulled tag	Add LOCAL_MODEL_MAP, run ollama list
OPENAI_API_KEY is not set	Environment variable missing	export OPENAI_API_KEY=...
Push rejected with GH013 / secret scanning	A key is inside a committed file, often __pycache__	Revoke the key, add __pycache__/ to .gitignore, recommit without it
error: 1 with no detail	Bare except hides the traceback	Use the traceback.print_exc() change in 5.1
Datasets note

Some other benchmark datasets are hosted on Baidu, which requires a Chinese phone number to create an account. I could not get these. A few system-path assignments in those scripts also need to be removed. Stick to MATH.

Step 1 and step 2

Step 1 handles decomposition and the reasoning pipeline you run here. Step 2 reasons using the allocation results saved by the adapter (see the repo README). Get step 1 working first.