# Notes
## Setup
- Windows native (no WSL), Python 3.14.2, venv at .venv
- Needed Set-ExecutionPolicy RemoteSigned (CurrentUser) to activate venv
- Plain `pytest` fails to import logic_utils; `python -m pytest` works

## Phase 1 observations (my own play-testing)

General: the Developer Debug Info panel shows the state from BEFORE the latest guess is processed (it lags one step). Seen directly in Run 4 and Run 5. Fresh load starts at Attempts = 1. Panel values below are "as displayed."

### Run 1: Normal (secret = 85, range 1-100, 8 attempts allowed)
| # | Entered | Hint shown | Expected hint (my assumption) | Attempts | Score |
|---|---------|------------|-------------------------------|----------|-------|
| 1 | -50 | Go LOWER! | Reject as out of range | 1 | 0 |
| 2 | -50 | Go LOWER! | Reject as out of range | 2 | -5 |
| 3 | 0 | Go LOWER! | Reject as out of range | 3 | -10 |
| 4 | 25 | Go LOWER! | Go HIGHER! | 4 | -15 |
| 5 | 100 | Go LOWER! | Go LOWER! | 5 | -20 |
| 6 | 86.2 | Go HIGHER! | Go LOWER! (or reject the decimal) | 6 | -25 |
| 7 | 85 | Correct! | Correct! | 7 | -30 |

Win message after #7: "You won! The secret was 85. Final score: -20" (panel said -30; the win adds +10, app.py:52-54).

Other observations (Run 1):
- Fresh load, before any guess: Attempts = 1, "Attempts left: 7", sidebar says 8 allowed.
- -50 and 0 were accepted and used up attempts, though the range is 1-100.
- 86.2 was accepted, but History stored 86 (decimal silently dropped).
- History showed 6 entries after 7 submissions (85 missing).
- Score dropped 5 per guess.

### Run 2: Easy (secret = 49, sidebar range 1-20, 6 attempts allowed)
Switched Normal -> Easy; UNCONFIRMED whether I clicked New Game afterwards.
| # | Entered | Hint shown | Expected hint | Attempts (panel) | Score (panel) |
|---|---------|------------|---------------|------------------|---------------|
| 1 | 1 | Go LOWER! | Go HIGHER! | 1 | 0 |
| 2 | 1 | Go LOWER! | Go HIGHER! | 2 | -5 |
| 3 | 25 | Go LOWER! | Go HIGHER! | 3 | -10 |
| 4 | -10 | Go LOWER! | Reject as out of range | 4 | -15 |
| 5 | 101 | Go LOWER! | Go LOWER! (or reject) | 5 | -20 |

Result: "Out of attempts! The secret was 49. Score: -25" (panel said -20).
- Secret 49 is outside the Easy range 1-20.
- Banner said "between 1 and 100" on Easy; sidebar said 1 to 20.
- Game ended after 5 guesses with 6 allowed (panel attempts hit 6 after guess 5).
- History showed 4 entries after 5 submissions.

### Run 3: Hard, New Game clicked after the Easy loss
- Sidebar: range 1-50, 5 attempts. Banner said "between 1 and 100. Attempts left: 5".
- After New Game, panel showed: Secret 37, Attempts 0, Score -25 (not reset), History still [1, 1, 25, -10, 101] (not reset).
- Screen showed "Game over. Start a new game to try again." even after New Game.
- Submit Guess did nothing; after several clicks a script indicator flashed briefly.
- A browser refresh made the game playable again.

### Run 4: Hard after browser refresh (secret = 84, sidebar range 1-50, 5 attempts allowed)
Before any guess: banner "between 1 and 100. Attempts left: 4"; panel Attempts 1, Score 0, History [].
| # | Entered | Hint shown | Expected hint | Attempts (panel) | Score (panel) |
|---|---------|------------|---------------|------------------|---------------|
| 1 | 7 | Go LOWER! | Go HIGHER! | 1 | 0 |
| 2 | -1000 | Go LOWER! | Reject as out of range | 2 | -5 |
| 3 | 1000 | Go LOWER! | Reject as out of range | 3 | -10 |
| 4 | 7 | Go LOWER! | Go HIGHER! | 4 | 0 |
| 5 | 99 | Go HIGHER! | Go LOWER! | 4 | -15 |

- Secret 84 is outside the Hard range 1-50 (unconfirmed whether difficulty was switched after load).
- Rows 4-5 DON'T FIT the code as I read it: Hard allows 5 attempts and app.py:182 ends the game when attempts >= limit, so the game should have ended after guess 4. Attempts repeating 4 and the score going 0 -> -15 are also unexplained. Maybe I clicked New Game or refreshed mid-run, or mis-transcribed. The hints themselves still fit the code.
- -1000 and 1000 were accepted and used attempts.

### Run 5: Normal, prediction test (secret = 19, 8 attempts allowed, fresh load, no New Game, no refresh)
| # | Entered | Hint shown | Expected hint | Attempts (panel) | Score (panel) |
|---|---------|------------|---------------|------------------|---------------|
| 1 | 35 | Go HIGHER! | Go LOWER! | 1 | 0 |
| 2 | 100 | Go HIGHER! | Go LOWER! | 2 | 5 |
| 3 | 100 | Go LOWER! | Go LOWER! | 3 | 0 |

- Same input (100) gave HIGHER on guess #2 and LOWER on guess #3 (and LOWER on guess #5 in Run 1). Matches the prediction from Claude's explanation.
- First positive score I've seen (+5 after guess 1).
- My "Attempts: 0" at the start was a misread: the final screenshot (panel Attempts 3 with 2 history entries, banner "left: 5") fits a start of 1.
- Hand trace (guess k judged at attempts = k+1): guess 35 at 2 (text compare), 100 at 3 (number compare), 100 at 4 (text compare). All three hints and the scores fit.

### pytest
Plain `pytest` (venv active), trimmed:
```
collected 0 items / 1 error
ERROR collecting tests/test_game_logic.py
tests\test_game_logic.py:1: in <module>
    from logic_utils import check_guess
E   ModuleNotFoundError: No module named 'logic_utils'
1 error in 0.57s
```
`python -m pytest` (venv active), trimmed:
```
collected 3 items
tests\test_game_logic.py FFF                                   [100%]
FAILED tests/test_game_logic.py::test_winning_guess - NotImplementedError: Refactor this function from app.py into logic_utils.py
FAILED tests/test_game_logic.py::test_guess_too_high - NotImplementedError: Refactor this function from app.py into logic_utils.py
FAILED tests/test_game_logic.py::test_guess_too_low - NotImplementedError: Refactor this function from app.py into logic_utils.py
3 failed in 0.43s
```
All three fail at logic_utils.py:21 (the stub). Plain pytest doesn't put the repo root on the import path; `python -m pytest` does.

### Claude explanation: wrong hints (session "app.py hints bug investigation", Plan mode, app.py + logic_utils.py attached)
Prompt: asked it to explain step by step what the code does on submit, where the wrong hint comes from (file and line), whether it verified by running or reading, and to use an analogy.

What Claude said:
- It read the code and confirmed line numbers with a search. It did NOT run anything; its traces were by hand.
- Bug A: hint text is backwards. app.py:37 checks guess > secret (too high), but line 38 says "Go HIGHER!"; line 40 is the reverse. Repeated at lines 46-47.
- Bug B: app.py:158-161 makes the secret a string on even attempts (str(secret)). int > str raises TypeError, handled at app.py:41, which compares guesses as text, so "100" > "85" is False ("1" sorts before "8").
- Analogy: sorting files by name instead of size.
- Claimed my "correct" 100 -> "Go LOWER!" was luck: on an odd attempt, 100 should show "Go HIGHER!".
- Its own stated uncertainty: couldn't see which attempt number each of my guesses landed on; thinks the even/odd switch looks deliberately planted but can't confirm.

My checks:
- [x] Opened app.py lines 37-47 and 158-161: match Claude's description.
- Hand-traced all hints from Runs 1, 2, 4, 5 (swapped text + string comparison on alternating guesses, guess k judged at attempts = k+1). Runs 1 and 2 also match the real scores and win/loss messages. The opposite numbering fails (predicts HIGHER for 100 on guess #5, Run 1; I saw LOWER).
- [x] Prediction test (Run 5): 100 as guess #2 -> "Go HIGHER!" (wrong), as predicted; same 100 as guess #3 -> "Go LOWER!".
- Read update_score (app.py:57-60): "Too High" gives +5 on even attempts, -5 otherwise. Matches the scores I saw. Win bonus (app.py:52-54): points = 100 - 10*(attempt_number+1), minimum 10.
- Read the New Game block (app.py:134-138) and lines 140-145: New Game resets only attempts (135) and secret (136, always randint(1, 100)); the leftover "lost" status then hits st.stop() (145). Explains the stuck "Game over" state. Read-only; not yet run.

Status: confirmed by reading AND by my own prediction test.

### Claude's hypotheses (from the file-overview prompt): status
- [x] Wrong hints: swapped text (app.py:37-47) + string comparison on even attempts (app.py:158-161). Reproduced in Runs 1-5.
- [x] On even attempts the secret becomes a string: confirmed (Run 5 test + code lines).
- [x] Attempts counter starts at 1, "attempts left" off by one: fresh load shows Attempts 1, left = allowed - 1.
- [x] New Game resets attempts to 0 but not status, score or history: seen in Run 3; code at app.py:134-138.
- [ ] New Game ignores the difficulty range: code says randint(1, 100) at app.py:136 (read, not yet run).
- [x] Info banner hardcodes "1 and 100": seen on Easy and Hard (banner code not yet located).
- [x] "Too High" scoring awards +5 on even attempts: app.py:57-59, seen in Run 5.
- [x] Starter tests fail: confirmed with `python -m pytest`, for the reason Claude predicted (NotImplementedError from the stubs).
- [ ] Claude's claim that tests expect a bare string while app.py's check_guess returns a tuple: untested until the stubs are implemented.

## Chosen bugs for reflection.md section 1
1. Wrong hints: Bug A (swapped text) and Bug B (secret cast to string on even attempts)
2. New Game doesn't reset score/history and leaves the game stuck until refresh
3. Difficulty range mismatch: banner always says 1-100 on Easy and Hard; secrets 49 (Easy) and 84 (Hard) are outside the range (initial secret setup not located; New Game uses randint(1, 100))
Backups: attempts off by one; out-of-range guesses accepted.

## To confirm
- Run 4 rows 4-5 (see note): did I click New Game or refresh, or mis-transcribe?
- Whether the Easy/Hard secrets were generated before I switched difficulty.
- Where the first secret and the banner text are created in app.py.

## Git log
- Commit 1: CLAUDE.md + NOTES.md (setup and play-testing)
- Commit 2 (pending): reflection.md + NOTES.md (bug log and Claude hint analysis)