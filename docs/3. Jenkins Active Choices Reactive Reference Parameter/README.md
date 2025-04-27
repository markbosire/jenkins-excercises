# Jenkins Active Choices Reactive Reference Parameter

This project demonstrates how to use Jenkins **Active Choices Reactive Parameters** to dynamically populate a dropdown based on another parameter's selection, with proper handling of shell commands.

## Overview

The pipeline uses two parameters:
1. **`CATEGORY`**: A dropdown to select a command category (System Info, Networking, Files).
2. **`COMMAND`**: A reactive dropdown that populates with commands based on the selected `CATEGORY`.

Key features:
- Uses **Groovy scripts** to generate dynamic options.
- Avoids trailing commas in shell commands with **"Omit Value Field"**.
- Executes the selected command in the pipeline.

---

## Setup

### 1. Jenkins Parameters

#### `CATEGORY` (Active Choices Parameter)
- **Script**:
  ```groovy
  return ["System Info", "Networking", "Files"]
  ```
- **Type**: Single Select.

#### `COMMAND` (Active Choices Reactive Reference Parameter)
- **Script**:
  ```groovy
  def systemInfo = ["uname -a", "uptime", "whoami"]
  def networking = ["ip addr", "ping -c 4 8.8.8.8", "ss -tulwn"]
  def files = ["ls -lh", "pwd", "find . -type f"]

  def options = []
  switch(CATEGORY) {
      case "System Info": options = systemInfo; break
      case "Networking":  options = networking; break
      case "Files":       options = files;      break
      default:            options = ["echo 'Please select a category'"]
  }

  def html = "<select name='value'>"
  options.each { html += "<option value='${it}'>${it}</option>" }
  html += "</select>"
  return html
  ```
- **Type**: Formatted HTML.
- **Advanced Option**: ✅ `Omit value field` (critical to avoid trailing commas).

---

### 2. Pipeline (`Jenkinsfile`)
```groovy
pipeline {
    agent any
    stages {
        stage('Run Selected Command') {
            steps {
                script {
                    echo "Selected Category: ${params.CATEGORY}"
                    echo "Selected Command: ${params.COMMAND}"
                    sh "${params.COMMAND}"
                }
            }
        }
    }
}
```

---

## Why "Omit Value Field" Fixes Trailing Commas

### The Problem
- By default, Jenkins adds a hidden `<input name="value">` that stores selections as a **list** (e.g., `[whoami]`).
- When referenced in the pipeline (`${params.COMMAND}`), Jenkins converts the list to a string with commas (e.g., `whoami,`), breaking shell commands.

### The Fix
- **"Omit value field"** removes the hidden input, preventing list conversion.
- The `<select>` dropdown now passes the value directly as a **plain string** (e.g., `whoami`).

---

## Usage
1. **Build with Parameters**:
   - Select a `CATEGORY` (e.g., "System Info").
   - The `COMMAND` dropdown updates dynamically (e.g., shows `whoami`).
2. **Run Pipeline**:
   - The selected command executes in the `sh` step (e.g., runs `whoami`).

---

