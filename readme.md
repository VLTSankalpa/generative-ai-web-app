# Development Environment Setup

This guide will walk you through setting up a development environment using pyenv and pipenv on a Mac M1.

## Prerequisites

- Mac M1 running macOS
- Homebrew package manager installed
- Terminal access

## Step 1: Install Xcode Command Line Tools

Before we begin, make sure you have Xcode Command Line Tools installed:

```markdown
xcode-select --install
```

## Step 2: Install pyenv

Install pyenv using Homebrew:

```bash
brew update
brew install pyenv
```

Add pyenv to your shell configuration. Open your shell configuration file (e.g., ~/.zshrc or ~/.bash_profile) and add the following lines:

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

Restart your terminal or run `source ~/.zshrc` to apply the changes.

## Step 3: Install Python 3.11 using pyenv

Install Python 3.11 using pyenv:

```bash
pyenv install 3.11.0
pyenv global 3.11.0
```

Verify the installation:

```bash
python --version
pyenv versions
```

## Step 4: Install pipenv

If you already in conda environment, deactivate it:

```bash
conda deactivate
```

Install pipenv using pip:

```bash
pip install pipenv
```

## Step 5: Set up the project environment

Create a new directory for your project:

```bash
mkdir t-chains
cd t-chains
```

Create a `Pipfile` in the project directory with the following content:

```
[[source]]
url = "<https://pypi.org/simple>"
verify_ssl = true
name = "pypi"

[packages]
langchain = "==0.0.352"
openai = "==0.27.8"
python-dotenv = "==1.0.0"

[dev-packages]

[requires]
python_version = "3.11"
```

Install the dependencies:

```bash
pipenv install
```

Activate the virtual environment:

```bash
pipenv shell
```

## Step 6: Create the Python script

Create a file named `main.py` in your project directory with the following content:

```python
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain, SequentialChain
from dotenv import load_dotenv
import argparse

load_dotenv()

parser = argparse.ArgumentParser()
parser.add_argument("--task", default="return a list of numbers")
parser.add_argument("--language", default="python")
args = parser.parse_args()

llm = OpenAI()

code_prompt = PromptTemplate(
    input_variables=["task", "language"],
    template="Write a very short {language} function that will {task}."
)
test_prompt = PromptTemplate(
    input_variables=["language", "code"],
    template="Write a test for the following {language} code:\\n{code}"
)

code_chain = LLMChain(
    llm=llm,
    prompt=code_prompt,
    output_key="code"
)
test_chain = LLMChain(
    llm=llm,
    prompt=test_prompt,
    output_key="test"
)

chain = SequentialChain(
    chains=[code_chain, test_chain],
    input_variables=["task", "language"],
    output_variables=["test", "code"]
)

result = chain({
    "language": args.language,
    "task": args.task
})

print(">>>>>> GENERATED CODE:")
print(result["code"])

print(">>>>>> GENERATED TEST:")
print(result["test"])
```

## Step 7: Set up environment variables

Create a `.env` file in your project directory and add your OpenAI API key:

```
OPENAI_API_KEY=your_api_key_here
```

Replace `your_api_key_here` with your actual OpenAI API key.

## Step 8: Run the Python program

To run the program, use the following command:

```bash
python main.py --task "generate a fibonacci sequence" --language python
```

You can modify the `--task` and `--language` arguments as needed.

## Troubleshooting

- If you encounter any issues with environment variables or API keys, try exiting the shell and re-entering using the `pipenv shell` command.
- Anaconda users may find that Pipenv conflicts with their environment. If this happens, deactivate your conda environment before using pipenv.

## Note

Deprecation warnings about Langchain 0.1.0 and 0.2.0 should be ignored as we are not using these versions.