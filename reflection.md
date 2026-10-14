# Reflection

> NOTE: I did not have the handout's exact four questions, so these answers cover the usual ones. Rename the headings to match your handout.

## 1. Which tool found the most / most useful issues?
Pylint reported the most items (about 30), mostly style and naming, but also real bugs such as the mutable default argument. Bandit found the most serious ones: `os.system` injection, MD5, and `eval`. Flake8 mostly overlapped with Pylint but is fast and good for PEP 8 formatting.

## 2. What did the tools miss?
The hardcoded and printed Stripe secret was not flagged by any tool, because Bandit only matches certain variable names. Also missed were the logic bugs: `total_sales` is never reduced when an item is removed, and `%d` silently truncates prices (35.99 logged as 35). Running the program and reading the output, plus code review, is what catches these.

## 3. Which fixes were hardest or most important, and why?
Replacing `eval` was the most important: it allowed arbitrary code execution from user input. The fix needed a design change (a numeric percentage with validation) rather than a one-line patch. Removing `os.system` was easy but just as critical, since filenames could inject shell commands.

## 4. How would you use static analysis in a real project?
Run Flake8, Pylint and Bandit in CI (pre-commit hooks or GitHub Actions) so every pull request is checked automatically, and fail the build on High severity Bandit findings. Pair it with code review, tests and secret scanning, because static tools produce false positives and miss logic and business-rule bugs.
