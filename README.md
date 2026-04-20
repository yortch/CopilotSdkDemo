# CopilotSdkDemo

Small demos that exercise the [GitHub Copilot SDK](https://github.com/github/copilot-sdk) in both .NET and Python. Each demo is a minimal weather-assistant chatbot that registers a `get_weather` tool and streams model responses.

For SDK concepts, authentication, and architecture, see the official [Getting Started Guide](https://github.com/github/copilot-sdk/blob/main/docs/getting-started.md).

## Prerequisites

- A GitHub account with Copilot access (or a BYOK API key — see the [auth docs](https://github.com/github/copilot-sdk/blob/main/docs/auth/index.md))
- [.NET 10 SDK](https://dotnet.microsoft.com/download) for the .NET demo
- [Python 3.10+](https://www.python.org/downloads/) for the Python demo

The Copilot CLI is bundled automatically with both SDKs — no separate install required.

## Repository layout

```
DotNetDemo/    # C# demo (CopilotSdkDemo.csproj / Program.cs)
PythonDemo/    # Python demo (main.py / requirements.txt)
```

## Running the .NET demo

```bash
cd DotNetDemo
dotnet run
```

## Running the Python demo

From the repository root:

```bash
# 1. Create and activate a virtual environment (first time only)
python -m venv .venv

# 2. Install dependencies
.venv/Scripts/pip.exe install -r PythonDemo/requirements.txt

# 3. Run the demo
.venv/Scripts/python.exe PythonDemo/main.py
```

On macOS/Linux, replace `.venv/Scripts/` with `.venv/bin/`.

## Using the demos

Once running, either demo prints a prompt. Try:

- `What's the weather in Paris?`
- `Compare the weather in NYC and LA`

Type `exit` to quit.

## References

- [GitHub Copilot SDK — Getting Started](https://github.com/github/copilot-sdk/blob/main/docs/getting-started.md)
- [Copilot SDK repository](https://github.com/github/copilot-sdk)
- [Authentication options (OAuth, BYOK, env vars)](https://github.com/github/copilot-sdk/blob/main/docs/auth/index.md)
- [Cookbook & recipes](https://github.com/github/awesome-copilot/blob/main/cookbook/copilot-sdk)
