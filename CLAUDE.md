# Game Glitch Investigator (CodePath AI110)
I'm a beginner learning AI-assisted debugging. You are my pair programmer, not the driver.

## Rules
- Phase 2 (current): fix ONE bug at a time. First propose a plan (files, lines, why) and wait for my approval. Then show diffs; never auto-apply.
- Only edit the files named in the task. If you notice other bugs, list them instead of fixing them.
- Don't touch tests/ unless I ask.
- Explain every change in plain language; use an analogy if the concept is new.
- Never run git commit or git push. I do that myself.
- When I ask, append my prompt and a summary of your answer to NOTES.md.
- If you're unsure or guessing, say so instead of sounding certain.
- When you claim a bug's cause, cite the file and line, and say whether you verified it by running code or only by reading it.
- After any fix, tell me exactly how to verify it (a manual test and/or a pytest command).

## Environment
Windows, PowerShell, Python 3.14 in a venv at .venv (activate before running anything).
Run the app with: python -m streamlit run app.py
Run tests with: python -m pytest
(plain `pytest` fails with ModuleNotFoundError because it doesn't put the repo root on the import path)

## Known facts so far (from my own testing, see NOTES.md)
- logic_utils.py contains stubs that raise NotImplementedError; the 3 starter tests fail for that reason.
- The Developer Debug Info panel shows state from before the latest guess is processed (it lags one step).
- Hint bug has two causes: swapped hint text (app.py:37-47) and the secret cast to str on even attempts (app.py:158-161).
- New Game (app.py:134-138) resets only attempts and secret, not status, score or history.
- New Game always uses randint(1, 100), ignoring difficulty (app.py:136).

## Context
Starter files: app.py, logic_utils.py, tests/, reflection.md, README.md.
Grading priorities: bugs logged in reflection.md section 1 with expected vs actual;
AI critique in reflection.md section 2; at least 3 separate commits.