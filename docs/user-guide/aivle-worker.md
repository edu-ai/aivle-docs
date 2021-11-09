# Validating Sandbox

[GitHub repo](https://github.com/edu-ai/aivle-worker)

---

1. Create `.env` (if you only intend to test sandbox, put dummy values on the RHS is enough):
```dotenv
BROKER_URI=amqp://...
ACCESS_TOKEN=...
TASK_QUEUE=gpu/private/default
```

2. Under `__main__.py`, rewrite paths in `start_sandbox()` function, run that function.

**NOTE**:

* There are three `/` in the URL (i.e. `file:///home/...`)
* You need to use absolute path

## Details

Referring to `settings.py`, by default the sandbox will load security profile from `profiles/aivle-base.profile`
and the shell script to create new virtual environment will be `scripts/create-venv.sh`, and the temporary folder will
be `/tmp/aivle-worker`. You don't need to change the settings if you're happy with the defaults.

The basic steps aivle Worker takes to evaluate an agent in your task bundle are:

1. Download and unzip task and agent archives
2. Create a new virtual environment (and activate it)
3. `pip install -r requirements.txt` (therefore in the root of your task bundle there must be a `requirements.txt`)
4. Inside the sandbox, run `bash bootstrap.sh` (similarly, `bootstrap.sh` comes from your task bundle)