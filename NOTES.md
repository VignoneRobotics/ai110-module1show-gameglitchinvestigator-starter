# Notes
## Setup
- Windows native (no WSL), Python 3.14.2, venv at .venv
- Needed Set-ExecutionPolicy RemoteSigned (CurrentUser) to activate venv
- Plain `pytest` fails to import logic_utils; `python -m pytest` works

## Phase 1 observations (my own play-testing)

General: the Developer Debug Info panel shows the state from BEFORE the latest guess is processed (it lags one step). Seen directly in Run 4: after guess #1 the hint appeared but the panel still showed Attempts 1, Score 0, History []. Panel values below are "as displayed."

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

Win message after #7: "You won! The secret was 85. Final score: -20" (debug panel said -30).

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
| 5 | 101 | Go LOWER! | Go LOWER! (or reject) | 5 (screenshot; my typed note said 4, recheck) | -20 |

Result: "Out of attempts! The secret was 49. Score: -25" (panel said -20).
- Secret 49 is outside the Easy range 1-20.
- Banner said "between 1 and 100" on Easy; sidebar said 1 to 20.
- Game ended after 5 guesses with 6 allowed (banner showed "Attempts left: 1").
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
- Score jumped from -10 to 0 on guess #4, then dropped to -15 on #5 (unexplained).
- Attempts stayed at 4 across guesses #4 and #5.
- -1000 and 1000 were accepted and used attempts.

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
All three fail at logic_utils.py:21 (the stub). Plain pytest doesn't put the repo root on the import path; `python -m pytest` does (this fixed it, which matches my guess).

### Claude's hypotheses (from the file-overview prompt): status after my testing
- [x] Wrong hints: reproduced in every run. "Swapped" alone doesn't fit (guesses of 100 and 101 above the secret got "Go LOWER!", which is correct; 86.2 and 7 got the wrong hint). Cause not confirmed.
- [ ] On even attempts the secret becomes a string: consistent with my data (large/multi-digit guesses behave oddly), not confirmed.
- [x] Attempts counter starts at 1, "attempts left" off by one: fresh load shows Attempts 1, left = allowed - 1.
- [x] New Game resets attempts to 0 but not status, score or history: seen in Run 3.
- [ ] New Game ignores the difficulty range: inconclusive (37 fits both 1-50 and 1-100).
- [x] Info banner hardcodes "1 and 100": seen on Easy and Hard.
- [ ] "Too High" scoring awards +5 on even attempts: not clearly seen (Run 4 score jump of +10 unexplained).
- [x] Starter tests fail: confirmed with `python -m pytest`, for the reason Claude predicted (NotImplementedError from the stubs).
- [ ] Claude's claim that tests expect a bare string while app.py's check_guess returns a tuple: untested until the stubs are implemented.

## Chosen bugs for reflection.md section 1
1. Wrong hints (Runs 1-4)
2. New Game doesn't reset score/history and leaves the game stuck until refresh (Run 3)
3. Difficulty range mismatch: banner always says 1-100 on Easy and Hard (certain, Runs 2-4); secret appears to ignore the range (suspected: 49 on Easy, 84 on Hard)
Backups: attempts off by one; out-of-range guesses accepted.

## To confirm
- Easy run row 5 attempts: 4 or 5 (screenshot says 5).
- Whether the Easy/Hard secrets were generated before I switched difficulty.

## Git log
- Commit 1: CLAUDE.md + NOTES.md (setup and play-testing)