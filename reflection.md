# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
  The page loaded fine, with a difficulty dropdown, a guess box, and a Developer Debug Info panel that shows the secret. On my first game (secret 85) I typed -50 and was told "Go LOWER!", which made no sense because I was already far below the secret, and the game accepted -50 and 0 as guesses even though the range is 1 to 100. When I finally guessed 85, the debug panel said my score was -30 but the win message said -20. Overall it looked like a working game whose hints, scoring, and reset behavior couldn't be trusted.

- Concrete bugs noticed at the start:
  1. **Wrong hints.** Guesses below the secret often got "Go LOWER!" (e.g. 25 vs secret 85; 7 vs 84), and guesses above it often got "Go HIGHER!" (e.g. 35 vs secret 19). The same input could even give different hints depending on the attempt number (100 vs secret 85 showed "Go LOWER!" on one attempt, while 100 vs secret 19 showed "Go HIGHER!").
  2. **New Game doesn't fully reset.** After a loss, New Game gave a new secret and reset attempts to 0, but Score (-25) and History carried over, and the game showed "Game over" and ignored guesses until I refreshed the browser.
  3. **Difficulty range mismatch.** The banner always says "between 1 and 100," while the sidebar says 1 to 20 (Easy) or 1 to 50 (Hard). Secrets of 49 on Easy and 84 on Hard suggest the secret ignores the range too (suspected, not confirmed).

**Bug Reproduction Log**
(Line numbers refer to the original starter code, before my edits.)

| Input | Expected Behavior | Actual Behavior | Console Output / Error | Suspected Code Location |
|-------|-------------------|-----------------|------------------------|-------------------------|
| Normal, secret 85, guess 25 | "Go HIGHER!" | "Go LOWER!" | none | app.py:37-41 (hint text swapped) |
| Normal, secret 19, guess 35, then guess 100 | "Go LOWER!" both times | "Go HIGHER!" both times | none | app.py:37-41 (hint text swapped); app.py:158-161 (secret cast to string on alternating attempts) |
| Hard, secret 84, guess 7; Easy, secret 49, guess 1 | "Go HIGHER!" | "Go LOWER!" | none | app.py:37-41; app.py:158-161 |
| Lose a game on Easy, switch to Hard, click New Game, then guess | Fresh game: score 0, empty History, guessing works | Secret and attempts reset, but Score stayed -25, History kept old guesses, "Game over" showed, Submit did nothing until a page refresh | none | app.py:134-138 (New Game resets only attempts and secret); app.py:140-145 (leftover "lost" status shows "Game over" and calls st.stop()) |
| Select Easy (or Hard) | Banner says "between 1 and 20" ("1 and 50"), secret within that range | Banner says "between 1 and 100"; secrets were 49 (Easy) and 84 (Hard) | none | app.py:136 (New Game uses random.randint(1, 100) regardless of difficulty); banner text and initial secret setup not yet located |

**Run trace (Normal difficulty, secret = 85, 8 attempts allowed)**
Attempts and Score are as shown in the Developer Debug Info panel, which lags one step.
```
guess  -50 -> "Go LOWER!"   attempts 1, score 0
guess  -50 -> "Go LOWER!"   attempts 2, score -5
guess    0 -> "Go LOWER!"   attempts 3, score -10
guess   25 -> "Go LOWER!"   attempts 4, score -15   (expected HIGHER)
guess  100 -> "Go LOWER!"   attempts 5, score -20
guess 86.2 -> "Go HIGHER!"  attempts 6, score -25   (expected LOWER)
guess   85 -> "Correct!"    attempts 7, score -30
Win message: "You won! The secret was 85. Final score: -20" (panel said -30)
```

---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)?
  I used Claude Code (Sonnet 5.5) inside VS Code to read the code, explain bugs, and make edits, and I used a separate Claude chat to plan the work, review what Claude Code produced, and make sure I understood it. I set rules for Claude Code in a CLAUDE.md file (propose before editing, one change at a time, never commit, say whether it ran code or only read it).

- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result).
  Before any fix, I asked Claude Code to explain the wrong hints. It said there were two separate bugs: the hint messages were swapped (app.py:37-47), and on even attempts the secret was turned into a string (app.py:158-161), so numbers were compared alphabetically ("100" sorts before "85"). It also told me it had only read the code, not run it. I verified by reading those exact lines myself, by checking its theory against every hint I had recorded, and by testing its prediction in the game: the same guess of 100 gave "Go HIGHER!" on guess 2 and "Go LOWER!" on guess 3, which is what its explanation said would happen.

- Give one example of an AI suggestion you did not accept as written (including what the AI suggested, why you rejected or changed it, and how you verified your version). It does not have to be a suggestion that was wrong: over-engineered, out of scope, harder to read, or a poor fit for this codebase all count.
  When Claude proposed removing the leftover `except TypeError` branch, its diff contained a placeholder comment ("# FIX: ... (the FIX comment from Diff A moves here)") instead of a real one, and it had also swapped the arrow emoji without being asked. I approved the change but told Claude to write the actual comment text, then read the applied diff in the editor to make sure no placeholder was left. I also asked it to show that branch as a separate diff so I could decide on it, and I only accepted it after checking Claude's "dead code" claim myself (parse_guess always returns an int and the secret comes from randint).

  More generally, Claude's first overview of the project listed several possible bugs after only reading the code, and I treated them as unverified hypotheses: I wrote them in my notes unchecked and only counted each one as a bug after I reproduced it myself. For example, its comment that the Hard range might be intentionally easier I never logged, and its claim that "Too High" scoring awards +5 on even attempts I saw once (the +5 in one of my runs) but did not fix. Apart from those, I accepted its diffs after reading them.

---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?
  I wanted two kinds of evidence: an automated test and a manual check in the game. For the hints, pytest passes (4 passed) and I replayed the exact inputs that used to fail. On a fresh game with secret 10, guesses of 1, 100, 1, 100 gave "Go HIGHER!", "Go LOWER!", "Go HIGHER!", "Go LOWER!", so odd and even attempts now behave the same.

- Describe at least one test you ran (manual or using pytest)  
  and what it showed you about your code.
  The three starter tests only compare the outcome ("Win", "Too High", "Too Low"), so they pass even with the swapped hint text. That showed me the starter tests couldn't see my bug at all. I added `test_hint_message_direction`, which checks the message: guess 60 vs secret 50 must contain "LOWER", and guess 40 vs 50 must contain "HIGHER". The starter tests were also failing at first (the stubs raised NotImplementedError), and moving `check_guess` into `logic_utils.py` and updating them to unpack the (outcome, message) tuple made them pass.

```
platform win32 -- Python 3.14.2, pytest-9.1.1, pluggy-1.6.0
collected 4 items

tests\test_game_logic.py ....                                    [100%]

============================== 4 passed in 0.75s ==============================
```

- Did AI help you design or understand any tests? How?
  Yes. Claude Code wrote `test_hint_message_direction` and pointed out that the old tests expected a bare string while `check_guess` returns a tuple. I decided to change the tests rather than the function, because the docstring and the only caller both use the tuple, and I made sure no assertion got weaker.

---

## 4. What did you learn about Streamlit and state?

- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?
  Every time you click a button or type in a box, Streamlit runs the whole script again from the top, like re-reading a recipe from step one each time. Normal variables get forgotten on each re-run, so anything the game has to remember (the secret, attempts, score, history, whether the game is over) is stored in `st.session_state`, which works like a notebook that survives between re-runs. My New Game bug is a good example: that button only wrote new values for two entries in the notebook (attempts and the secret), so the old score, history, and "lost" status were still there on the next re-run and the game stopped itself right away.

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
  Playing the game first and writing down exact inputs, hints, and scores before asking any AI anything. That gave me evidence to check Claude's explanation against (it matched every hint I had recorded, and its prediction that the same guess of 100 would give different hints on different attempts came true). I also want to keep committing after each working step.
- What is one thing you would do differently next time you work with AI on a coding task?
  I clicked "use auto mode" on one of Claude's plan prompts, so its edits were applied before I saw them. The diff turned out to be fine, but I only checked it afterwards. Next time I'll check the mode label before every prompt and approve each edit manually, especially at the start of a task.
- In one or two sentences, describe how this project changed the way you think about AI generated code.
  AI-written code can look reasonable and still be wrong in ways that only show up with particular inputs (the same guess gave different hints on different attempts), and the tests it comes with can pass while the bug is still there. AI is a good partner for explaining and fixing, but I now treat its claims as hypotheses until I've checked them against what the program really does.