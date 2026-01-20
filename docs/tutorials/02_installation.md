# Tutorial 2: Installation

Getting LazyHooks running is easy.

## Prerequisites

You need **Python 3.8** or higher installed on your computer.
To check if you have it, open your terminal (Command Prompt or PowerShell on Windows) and type:

```bash
python --version
```

If you see `Python 3.x.x`, you are good to go!

## Installing LazyHooks

## Pro Tip: Virtual Environments 🛡️

Python projects can conflict with each other. To keep things clean, we create a "Virtual Environment" (a sandbox) for each project.

**Mac/Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```

**Windows:**
```powershell
python -m venv venv
.\venv\Scripts\activate
```

*(You'll know it worked because you'll see `(venv)` in your terminal prompt!)*

## Installing LazyHooks

Now that you are in your sandbox (or if you chose to skip that step), install the package:

Run this command in your terminal:

```bash
pip install lazyhooks
```

## Verifying It Works

Let's make sure it installed correctly. Run this simple one-liner in your terminal:

```bash
python -c "import lazyhooks; print(lazyhooks.__version__)"
```

If it prints a version number (like `0.2.0`), congratulations! You are ready to write code.

[Next: Tutorial 3 - Your First Webhook ->](03_your_first_webhook.md)
