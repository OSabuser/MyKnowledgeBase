---
title: "VScode: Defining own variables in tasks.json"
source: "https://stackoverflow.com/questions/44303316/vscode-defining-own-variables-in-tasks-json"
author:
  - "[[Stack Overflow]]"
published: 2017-06-01
created: 2025-02-13
description: "I've set up a tasks.json file for building a project on multiple platforms. All platforms see the same content of the project repository. This is done either via disk sharing, because of running an..."
tags:
  - "clippings"
---
I am taking a different approach.

### tasks.json

```json
{
    "version": "2.0.0",
    "params":{
        "git_version":"2.30.0",
        "node_version":"14.13.6",
        "python_version":"3.8"
    },
    "tasks": [
        {
            "label":"Process Task.json",
            "type":"shell",
            "command":"python process_tasks.py",
            "group":"build",
            "isBackground":true
        },
        {
            "label":"Test process_tasks.py",
            "type":"shell",
            "command":"echo $[params.git_version]",
            "group":"test",
            "presentation": {
                "reveal": "always"
            }
        }
     ]
}
```

Rather than making env variables, we can follow the flowing steps:

### Step 1:

Make a task in tasks.json as follows

```json
{
    "label":"Process Task.json",
    "type":"shell",
    "command":"python process_tasks.py",
    "group":"build",
    "isBackground":true
},
```

### Step 2:

process\_tasks.py is a Python file that will replace variables in tasks.json with the actual value:

```py
import json
import os
import re

if __name__ == "__main__":
    # Get the path to the JSON file
    json_path = os.path.join(".vscode/tasks.json")

    with open(json_path, "r") as f:
        data = json.load(f)

    with open(json_path, "r") as f:
        lines = f.readlines()

        new_lines = []

        #regex to find text between $[]
        regex = re.compile(r"\$\[(.*?)\]")

        for line in lines:
            #regex in line:
            match = regex.search(line)
            if match:
                #get the text between $[]

                text = match.group(1)

                keys = text.split(".")

                buffer_data = data

                for key in keys:
                    buffer_data = buffer_data[key]

                #replace the text with the value of the environment variable
                line = line.replace(f"$[{text}]", buffer_data)

            new_lines.append(line)

    with open(json_path, "w") as f:
        f.writelines(new_lines)
```

### Step 3:

Add a test task to verify your result

```json
{
    "label":"Test process_tasks.py",
    "type":"shell",
    "command":"echo $[params.git_version]",
    "group":"test",
    "presentation": {
        "reveal": "always"
    }
},
```

### Note:

Making this "Process Task.json" as a global task and adding the correct path of the process.py file in the build task will reduce a lot of work.

Thus we can define our own variables inside tasks.json and access them using `$[params.git_version]`.

After executing the "Process Task.json" task, all variables in `$[]` format will be replaced by its corresponding value.