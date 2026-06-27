# passgen — Local-first Secure Password Generator

A cryptographically secure password generator and evaluator that runs entirely in your browser. No servers. No accounts. No data collection. Everything stays on your machine.

---

## Why "local-first"?

Most online password tools send your data somewhere — to a server that generates the password, to analytics that log your session, or to a cloud backend that stores your history. **passgen does none of that.**

- Passwords are generated using the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues) (`crypto.getRandomValues`), which calls your operating system's cryptographically secure random number generator directly in the browser.
- Your password history is stored exclusively in your browser's `localStorage`. It never leaves your machine, is never sent over the network, and is never accessible to anyone but you.
- The app is a single `index.html` file. You can open it from your hard drive with no internet connection and it works exactly the same.

---

## Features

**Generation**
- Cryptographically secure randomness via `crypto.getRandomValues` — not `Math.random()`
- Guaranteed character diversity: at least one character from each selected group is always present
- Cryptographic shuffle (Fisher-Yates with `crypto.getRandomValues`) so required characters never cluster at the start
- Configurable length from 4 to 64 characters
- Four independent character groups: uppercase, lowercase, digits, special characters

**Evaluation**
- Real-time strength meter based on six concrete criteria
- Entropy estimate in bits — the actual metric security systems use
- Works on passwords you didn't generate here: paste any password into the Evaluate tab

**History**
- Every generated password is saved to `localStorage` with a timestamp
- Persists across page reloads and browser restarts
- Capped at 30 entries to avoid accumulating sensitive data indefinitely
- One-click clear wipes the entire history from `localStorage`

**Privacy**
- Zero network requests for core functionality
- No cookies, no tracking, no third-party scripts
- Optional AI analysis (the `+ AI` button) makes a single API call to Anthropic with only metadata — character count, entropy, and criteria score — never the password itself

---

## How localStorage is used

```js
const STORAGE_KEY = 'passgen_history';

// Save
localStorage.setItem(STORAGE_KEY, JSON.stringify(history));

// Load on startup
JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];

// Clear
localStorage.removeItem(STORAGE_KEY); // called on "clear history"
```

The stored value is a JSON array of objects with two fields: the password string and a timestamp. It lives under the `file://` or `localhost` origin, meaning no other website can read it. To inspect or delete it manually, open DevTools → Application → Local Storage.

---

## Why `crypto.getRandomValues` instead of `Math.random()`

| | `Math.random()` | `crypto.getRandomValues()` |
|---|---|---|
| Source | Pseudorandom algorithm (V8 xorshift128+) | OS-level CSPRNG (`/dev/urandom`, `BCryptGenRandom`) |
| Predictable if seeded | Yes | No |
| Suitable for passwords | No | Yes |
| Standard for security use | No | Yes (NIST SP 800-90A) |

`Math.random()` is fast and fine for games or UI animations. It is not appropriate for generating secrets, because its output can be predicted if an attacker observes enough values. `crypto.getRandomValues()` is the browser equivalent of Python's `secrets` module.

---

## Entropy reference

Entropy measures how many bits of randomness a password contains. Higher entropy = exponentially harder to brute-force.

| Entropy | Resistance |
|---|---|
| < 40 bits | Weak — crackable in seconds with modern hardware |
| 40–60 bits | Moderate — reasonable for low-stakes accounts |
| 60–80 bits | Good — suitable for most personal accounts |
| > 80 bits | Very strong — appropriate for high-value targets |

A 16-character password using all four character groups yields roughly 105 bits of entropy.

---

## Getting started

No installation, no dependencies, no build step.

```bash
git clone https://github.com/YOUR_USERNAME/passgen.git
cd passgen
open index.html        # macOS
xdg-open index.html    # Linux
start index.html       # Windows
```

Or just download `index.html` and double-click it.

---

## Repository structure

```
passgen/
├── index.html   # The entire web app — HTML, CSS, and JS in one file
├── main.py      # CLI version using Python's secrets module
└── README.md
```

---

## Tech stack

**Web app (`index.html`)**
- Vanilla HTML, CSS, JavaScript — no frameworks, no bundler
- Web Crypto API for generation
- `localStorage` for persistence
- IBM Plex Mono + Inter via Google Fonts

**CLI (`main.py`)**
- Python 3.6+
- `secrets` module (stdlib) — same CSPRNG guarantee as the web version
- `math` module for entropy calculation
- Zero external dependencies

---

## Security notes

- This tool generates passwords. It does not store, transmit, or manage them. Use a dedicated password manager (Bitwarden, 1Password, KeePass) to store passwords securely.
- The `localStorage` history is convenient for quickly re-copying a just-generated password, but it is not encrypted. Do not leave sensitive passwords in the history on a shared machine. Use the clear button when done.
- The optional AI analysis sends only statistical metadata to the Anthropic API. The password itself is never included in the request.

---

## License

MIT
