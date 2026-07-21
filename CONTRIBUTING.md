# Contributing

Thanks for taking a look. This project is deliberately small: one stdlib-only script, one agent contract (`SKILL.md`), one offline test suite. Please keep it that way.

## Ground rules

- **Single file, no new dependencies.** The script must stay a drop-in single file with Telethon as the only third-party import. No frameworks, no config files.
- **The JSON contract is sacred.** Exactly one JSON object on stdout, exit 0/1, empty stderr, never a traceback, no echo of environment values. Any change that can break this needs a test proving it does not.
- **Fail closed.** Validation errors refuse loudly before anything is written. A write that already happened must never be re-reported as a failure afterwards.
- **English only** in code, comments, docs, and commit messages.

## Running the tests

The offline suite needs no network, no credentials, and not even Telethon (a stub is injected; the few subprocess tests that need the real library skip themselves when it is absent):

```bash
python3 test_telethon_checklist.py
```

All 143 tests must pass with empty stderr. With `telethon>=1.44` installed, the integration test also verifies the exact MTProto request signatures.

Lint (the CI gate uses the critical rule set only):

```bash
ruff check --select E9,F63,F7,F82 telethon_checklist.py test_telethon_checklist.py
```

## Live smoke test (optional, your own account)

Follow [examples/quickstart.md](examples/quickstart.md) against Saved Messages (`me`): create -> get -> toggle -> append -> get, then delete the test list by hand. Never smoke-test against a chat with other people in it.

## Pull requests

- Small, focused changes; one concern per PR
- Every behavior change comes with a test that pins it (this codebase grew its suite adversarially - a fix without a regression test will be asked to add one)
- Run the full suite and the linter before pushing
- Never include session files, credentials, or personal chat ids in code, tests, or fixtures - use obviously fake ids like `-1001234567890`
