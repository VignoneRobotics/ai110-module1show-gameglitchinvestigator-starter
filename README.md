# 🎮 Game Glitch Investigator: The Impossible Guesser

## 🚨 The Situation

You asked an AI to build a simple "Number Guessing Game" using Streamlit.
It wrote the code, ran away, and now the game is unplayable.

- You can't win.
- The hints lie to you.
- The secret number seems to have commitment issues.

## 🛠️ Setup

1. Create and activate a virtual environment (Windows PowerShell: `python -m venv .venv` then `.\.venv\Scripts\Activate.ps1`)
2. Install dependencies: `pip install -r requirements.txt`
3. Run the app: `python -m streamlit run app.py`
4. Run the tests: `python -m pytest` (plain `pytest` couldn't import `logic_utils` on my setup)

## 🕵️‍♂️ Your Mission

1. **Play the game.** Open the "Developer Debug Info" tab in the app to see the secret number. Try to win.
2. **Find the State Bug.** Why does the secret number change every time you click "Submit"? Ask ChatGPT: *"How do I keep a variable from resetting in Streamlit when I click a button?"*
3. **Fix the Logic.** The hints ("Higher/Lower") are wrong. Fix them.
4. **Refactor & Test.**
   - Move the logic into `logic_utils.py`.
   - Run `pytest` in your terminal.
   - Keep fixing until all tests pass!

## 📝 Document Your Experience

- [x] **The game's purpose.** A number-guessing game built with Streamlit. The player picks a difficulty (Easy: 1 to 20 with 6 attempts, Normal: 1 to 100 with 8, Hard: 1 to 50 with 5), types guesses, and gets "Go HIGHER!" or "Go LOWER!" hints until they win or run out of attempts. The game tracks a score and a guess history, and a Developer Debug Info panel shows the secret number for testing.

- [x] **Bugs I found** (by playing the game and logging exact inputs and outputs; details are in `reflection.md`):
  1. **Wrong hints (fixed).** Two separate causes in the original code: the hint text was swapped ("Too High" said "Go HIGHER!"), and on every other attempt the secret was converted to a string, so numbers were compared alphabetically ("100" sorts before "85"). Result: the same guess could give different hints on different attempts.
  2. **New Game didn't fully reset (fixed).** After a win or loss, New Game reset the secret and attempts but left the old score, the old guess history, and the "won"/"lost" status, so the game said "Game over" and ignored guesses until I refreshed the page.
  3. **Difficulty range mismatch (not fixed).** The banner always says "between 1 and 100", and New Game picks the secret with `randint(1, 100)` regardless of difficulty. I also saw secrets of 49 on Easy and 84 on Hard, which is outside their ranges (suspected, I don't know exactly where the first secret is created).
  4. **Other issues I noticed but did not fix:** the attempts counter starts at 1 on page load but 0 after New Game; guesses outside the range (-1000, 1000, 101, -1) are accepted and use up attempts; "Too High" guesses on even attempts add +5 to the score while every other guess subtracts 5; and the Developer Debug Info panel shows the state one step behind the latest guess.
  5. **One bug from the mission text I did not see:** the mission says the secret changes every time you click Submit. In every game I played the secret stayed the same until New Game.

- [x] **Fixes I applied** (with Claude Code in VS Code, one change at a time, reviewing each diff):
  - Moved `check_guess` from `app.py` into `logic_utils.py` and imported it back, then updated the three starter tests to unpack its `(outcome, message)` return value (no assertions weakened).
  - Swapped the hint text (and arrows) so "Too High" says "Go LOWER!" and "Too Low" says "Go HIGHER!".
  - Removed the string conversion of the secret in `app.py`, so comparisons are always number vs number, and removed the now-unreachable `except TypeError` branch in `check_guess`.
  - Made the New Game button also reset score, history, and status.
  - Added `test_hint_message_direction`, which checks the hint message (the starter tests only checked the outcome, so they couldn't catch the swapped hints).
  - How I worked with the AI, including a suggestion I changed, is in `reflection.md`.

## 📸 Demo Walkthrough

Steps from my test sessions on Normal difficulty (sidebar: Range 1 to 100, Attempts allowed 8). The Developer Debug Info panel updates one step behind, so I read it after the next click.

Session A (secret 10 in the Debug Info panel, fresh page load):
1. User enters 1 → game shows "Go HIGHER!" (correct, 1 is below 10)
2. User enters 100 → "Go LOWER!"
3. User enters 1 again → "Go HIGHER!"; enters 100 again → "Go LOWER!" (the same hints on odd and even attempts; before the fix they flipped)
4. Score drops by 5 per guess: the panel showed 0, -5, -10, -15 after those four guesses

Session B (secret 52):
5. User enters 52 on the first guess → "Correct!" and "You won! The secret was 52. Final score: 70"
6. User clicks New Game → Score 0, History empty, and guessing works again without refreshing the page
7. User makes 8 wrong guesses in a row (3, 90, 45, 36, 37, 101, -1, 77 against secret 83) → the panel shows Attempts 8, Score -20, and a History of those 8 guesses; a further submit shows "Game over. Start a new game to try again."
8. User clicks New Game again → the game is playable again with no page refresh (before the fix this left the game stuck on "Game over")

**Screenshot** *(optional)*: not included

## 🧪 Test Results

```
platform win32 -- Python 3.14.2, pytest-9.1.1, pluggy-1.6.0
collected 4 items

tests\test_game_logic.py ....                                    [100%]

============================== 4 passed in 0.09s ==============================
```

## 🚀 Stretch Features

No stretch challenges completed.