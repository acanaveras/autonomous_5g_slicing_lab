# Automate 5G Network Lab Setup

### Automate 5G Network Configurations with NVIDIA AI LLM Agents and Kinetica Accelerated Database

This DLI introduces an Agentic Generative AI solution designed to address bandwidth allocation challenges in 5G networks. It is divided into two parts that guide you through setting up and managing a 5G network.

```python
!git clone https://github.com/acanaveras/autonomous_5g_slicing_lab.git

```

Insert your API Key in the next cell to get it stored for the application

```python
import yaml
from getpass import getpass
from pathlib import Path

# Prompt for API Key securely (input not shown)
api_key = getpass("Enter your API Key: ")

# Define the path to the YAML file
yaml_path = Path("./autonomous_5g_slicing_lab/agentic-llm/config.yaml")

# Read existing YAML if it exists, otherwise start fresh
if yaml_path.exists():
    with open(yaml_path, 'r') as file:
        try:
            config = yaml.safe_load(file) or {}
        except yaml.YAMLError:
            config = {}
else:
    config = {}

# Insert or update the API_KEY
config['API_KEY'] = api_key

# Write the updated configuration back to the YAML file
with open(yaml_path, 'w') as file:
    yaml.safe_dump(config, file)

print("✅ API Key successfully saved to config.yaml")

```

### Agentic LLMs for 5G Section

Once you have the **5G Lab GitHub** repository cloned, you can proceed to the **Agentic LLMs** section. This part of the lab demonstrates how to deploy an agentic workflow to monitor network performance and dynamically adjust bandwidth allocation.

- **Part A – Setup of 5G Lab environment**  
  Located at: `./autonomous5g_slicing_lab/llm-slicing-5g-lab/DLI_Lab_Setup.ipynb`  
  Provides instructions to set up a 5G Network Software Stack in your environemnt.

- **Part B – 5G Network Agent Workflow**  
  Located at: `./autonomous5g_slicing_lab/agentic-llm/agentic_pipeline_DLI.ipynb`  
  Explains the agentic pipeline in **LangGraph** for managing 5G network slicing and bandwidth allocation.

