# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
  The game loaded normally, with a difficulty selector, a guess box, and a Developer Debug Info panel. [REWRITE IN YOUR OWN WORDS: what you noticed on first play and what felt off.] The hints often pointed the wrong way, New Game didn't fully reset the game, and the on-screen range didn't match the difficulty I picked.

- Concrete bugs noticed at the start:
  1. **Wrong hints.** Guesses below the secret often got "Go LOWER!" (e.g. 25 vs secret 85; 7 vs 84), and guesses above it often got "Go HIGHER!" (e.g. 35 vs secret 19). The same input could even give different hints depending on the attempt number (100 vs secret 85 showed "Go LOWER!" on one attempt, while 100 vs secret 19 showed "Go HIGHER!").
  2. **New Game doesn't fully reset.** After a loss, New Game gave a new secret and reset attempts to 0, but Score (-25) and History carried over, and the game showed "Game over" and ignored guesses until I refreshed the browser.
  3. **Difficulty range mismatch.** The banner always says "between 1 and 100," while the sidebar says 1 to 20 (Easy) or 1 to 50 (Hard). Secrets of 49 on Easy and 84 on Hard suggest the secret ignores the range too (suspected, not confirmed).

**Bug Reproduction Log**

| Input | Expected Behavior | Actual Behavior | Console Output / Error | Suspected Code Location |
|-------|-------------------|-----------------|------------------------|-------------------------|
| Normal, secret 85, guess 25 | "Go HIGHER!" | "Go LOWER!" | none | app.py:37-41 (hint text swapped) |
| Normal, secret 19, guess 35, then guess 100 | "Go LOWER!" both times | "Go HIGHER!" both times | none | app.py:37-41 (hint text swapped); app.py:158-161 (secret cast to string on alternating attempts) |
| Hard, secret 84, guess 7; Easy, secret 49, guess 1 | "Go HIGHER!" | "Go LOWER!" | none | app.py:37-41; app.py:158-161 |
| Lose a game on Easy, switch to Hard, click New Game, then guess | Fresh game: score 0, empty History, guessing works | Secret and attempts reset, but Score stayed -25, History kept old guesses, "Game over" showed, Submit did nothing until a page refresh | none | (to find) |
| Select Easy (or Hard) | Banner says "between 1 and 20" ("1 and 50"), secret within that range | Banner says "between 1 and 100"; secrets were 49 (Easy) and 84 (Hard) | none | (to find) |

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
- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result).
- Give one example of an AI suggestion you did not accept as written (including what the AI suggested, why you rejected or changed it, and how you verified your version). It does not have to be a suggestion that was wrong: over-engineered, out of scope, harder to read, or a poor fit for this codebase all count.

---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?
- Describe at least one test you ran (manual or using pytest)  
  and what it showed you about your code.
- Did AI help you design or understand any tests? How?

---

## 4. What did you learn about Streamlit and state?

- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
- What is one thing you would do differently next time you work with AI on a coding task?
- In one or two sentences, describe how this project changed the way you think about AI generated code.