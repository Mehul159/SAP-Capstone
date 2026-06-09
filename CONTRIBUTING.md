# Contributing

Thanks for your interest in improving the SAP BDC Sales Revenue Analytics Platform.

## Development setup

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## Workflow

1. Create a feature branch from `main`:

   ```bash
   git checkout -b feature/short-description
   ```

2. Make your changes and keep commits focused and descriptive.
3. Run the linter and the test suite locally before pushing:

   ```bash
   ruff check .
   pytest --cov=etl
   ```

4. Open a pull request against `main`. CI must pass before merge.

## Coding standards

- Target Python 3.11+.
- Formatting and linting are enforced by [ruff](https://github.com/astral-sh/ruff); run
  `ruff check . --fix` to auto-fix where possible.
- Add or update unit tests in `tests/` for any behavioural change.
- Keep functions small and documented with concise docstrings.

## Commit messages

Use clear, imperative summaries (e.g. `Add QoQ growth KPI view`). Reference issues where relevant.
