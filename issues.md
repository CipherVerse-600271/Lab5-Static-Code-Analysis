# Issues Found - ecommerce_cart_system.py

Pylint: 3.86/10 before, 10.00/10 after. Bandit: 2 High, 1 Medium, 2 Low before, 0 after. Flake8: 7 warnings before, 0 after.

| Issue | Type | Line(s) | Description | Fix Approach |
|---|---|---|---|---|
| OS command injection via `os.system` | Security - High (Bandit B605) | 60 | The shell command is built by concatenating `filename`, so a name like `x & del file` runs extra commands. | Removed `os.system`; use `print()` to show the message. |
| Weak MD5 hash | Security - High (Bandit B324) | 49 | MD5 is cryptographically broken and unsafe for security use. | Switched to `hashlib.sha256`. |
| Hardcoded live payment secret, printed to console | Security - High (found by reading output, no tool flagged it) | 72, 81 | The Stripe key is in source code and leaked to stdout. | Read it from the `STRIPE_SECRET_KEY` environment variable; print only whether it is configured. |
| `eval()` on user-supplied discount string | Security - Medium (Bandit B307, Pylint W0123) | 44 | Any string passed as `discount_expr` is executed as Python code, allowing arbitrary code execution. | Removed `eval`; the function now takes a numeric `discount_percent`, validates 0-100 and calculates the price. |
| `total_sales` not updated on removal | Logic bug - Medium (no tool flagged it) | 12, 20, 25-29 | Items add to `total_sales` but removing one never subtracts, so the total drifts out of sync. It is also never used. | Removed the global; `get_total_sales()` calculates the total from the cart each time. |
| Mutable default argument `audit_log=[]` | Bug - Medium (Pylint W0102) | 15 | The same list is shared across all calls, leaking state between them. | Default is `None`; the caller passes its own list. |
| `%d` truncates the price in the audit log | Logic bug - Medium (no tool flagged it) | 21 | A price of 19.99 is logged as 19. | Use an f-string with `{price:.2f}`. |
| `random` used for the checkout PIN | Security - Low (Bandit B311) | 53 | The PRNG is predictable and not suitable for security. | Use `secrets.randbelow()`. |
| Bare `except: pass` | Security/Quality - Low (Bandit B110, Flake8 E722, Pylint W0702) | 26-29 | Silently hides all errors, and the caller can't tell whether the removal worked. | Use `dict.pop(item_id, None)` and return True/False. |
| File opened without `with` or encoding | Quality (Pylint R1732, W1514) | 57-59 | The file handle can leak if writing fails, and the encoding depends on the platform. | Use `with open(..., encoding="utf-8")`. |
| `main()` called on import | Design | 84 | Importing the module runs the whole demo. | Added `if __name__ == "__main__":`. |
| Naming and missing docstrings | Style (Pylint C0103, C0114, C0116) | many | camelCase function names and no docstrings violate PEP 8 / PEP 257. | Renamed to snake_case and added docstrings. |
| `== None` and if/else returning booleans | Style (Flake8 E711, Pylint C0121, R1703, R1705) | 34-39 | Not idiomatic Python. | `item is not None and item["qty"] > 0` in one return. |
| Unused import and variables | Dead code (Flake8 F401, F841; Pylint W0611, W0612) | 4, 33, 43, 65 | `math`, `handling_fee`, `base_price` and `k` are never used. | Removed them. |
| Parameter shadows global `cart_items` | Quality (Pylint W0621) | 63 | The parameter name hides the outer variable. | Renamed the parameter to `cart`; the global is `CART_ITEMS`. |
| Line too long, no final newline | Style (Flake8 E501, W292; Pylint C0304) | 76, 84 | Formatting violations. | Reformatted. |