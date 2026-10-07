# Game Glitch Investigator (CodePath AI110)
I'm a beginner learning AI-assisted debugging. You are my pair programmer, not the driver.

## Rules
- Phase 1 (current): observe and explain only. Do NOT edit any files.
- Before any edit: propose a plan and wait for my approval.
- One bug / one change at a time. Show diffs; never auto-apply.
- Explain every change in plain language; use an analogy if the concept is new.
- Don't touch tests/ unless I ask.
- Never run git commit or git push. I do that myself.
- When I ask, append my prompt and a summary of your answer to NOTES.md.
- If you're unsure or guessing, say so instead of sounding certain.
- When you claim a bug's cause, cite the file and line, and say whether you verified it by running code or only by reading it.

## Environment
Windows, PowerShell, Python 3.14 in a venv at .venv (activate before running anything).
Run the app with: python -m streamlit run app.py
Run tests with: python -m pytest
(plain `pytest` fails with ModuleNotFoundError because it doesn't put the repo root on the import path)

## Known facts so far (from my own testing, see NOTES.md)
- logic_utils.py contains stubs that raise NotImplementedError; the 3 starter tests fail for that reason.
- The Developer Debug Info panel shows state from before the latest guess is processed (it lags one step).

## Context
Starter files: app.py, logic_utils.py, tests/, reflection.md, README.md.
Grading priorities: bugs logged in reflection.md section 1 with expected vs actual;
AI critique in reflection.md section 2; at least 3 separate commits.