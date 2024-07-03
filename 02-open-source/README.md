## How to responds to questions

For this modele the questions and commands are

### Q1 Running Ollama with Docker

First execute this command
```bash
docker run -it \
    --rm \
    -v ollama:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

In other terminal executed this command
```bash
docker exec -it ollama bash
```

Then in the container execute this last command to get the `version`
```bash
ollama --version
```

### Q2 Downloading an LLM

While in the 2nd terminal running commands in the container executed this command
```bash
ollama pull gemma:2b 
```

Then changed the to the directory of the file like this 
```bash
cd /root/.ollama/models/manifests/registry.ollama.ai/library/gemma
```

Finally got the content of the file in the terminal with:
```bash
cat 2b
```

### Q3 Running the LLM

To run the gemma model in the terminal while in the container bash executed this other command:
```bash
ollama run gemma:2b
```

And finally executed the requested question in the interpreter to get the answer:
```bash
10 * 10
```

### Q4 Donwloading the weights

As requested executed this commands:
```bash
mkdir ollama_files

docker run -it \
    --rm \
    -v ./ollama_files:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

And 
```bash
docker exec -it ollama ollama pull gemma:2b 
```

The to get the answer executed this command in the 2nd terminal:
```bash
du -sh ollama_files/
```

### Q5 Adding the weights

Create a docker file with:
```Dockerfile
FROM ollama/ollama

COPY ollama_files /root/.ollama
```

### Q6 Serving it

Execute the requested commands to build the docker image with the `Dockerfile` created previously:
```bash
docker build -t ollama-gemma2b .
```

Then create a container using this image:
```bash
docker run -it --rm -p 11434:11434 ollama-gemma2b
```
Now to execute the client request with python, install all this dependencies
```bash
pip install requests jupyter elasticsearch tqdm openai tiktoken
```

After that executed this lines in a Jupyter Notebook:
```python
# Create client connected to local ollama instance
from openai import OpenAI

client = OpenAI(
    base_url='http://localhost:11434/v1/',
    api_key='ollama',
)

# set the prompt and execute the query to the model used
# and using the temperature requested
prompt = "What's the formula for energy?"

response = client.chat.completions.create(
    model='gemma:2b',
    messages=[{"role": "user", "content": prompt}],
    temperature=0.0
)
# print the response to get the completion_tokens from the final response object
response
```

