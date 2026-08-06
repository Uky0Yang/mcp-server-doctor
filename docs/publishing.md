# Publishing

`mcp-server-doctor` is published on PyPI through Trusted Publishing and GitHub Actions.

## PyPI Trusted Publisher

The trusted publisher is configured with:

- PyPI project name: `mcp-server-doctor`
- Owner: `Uky0Yang`
- Repository: `mcp-server-doctor`
- Workflow name: `release.yml`
- Environment name: `pypi`

Pushing a version tag runs `.github/workflows/release.yml`, builds the wheel and source distribution, checks them with Twine, and publishes to PyPI through OIDC.

```bash
git tag vX.Y.Z
git push origin vX.Y.Z
```

The first Trusted Publishing release, `v0.2.0`, was published successfully. Before creating a new tag, update the package version and changelog, run the full test and build checks, and confirm the tag does not already exist.

## Why Trusted Publishing

Trusted Publishing avoids long-lived PyPI API tokens in GitHub secrets. PyPI mints a short-lived token only for the configured repository, workflow, and environment.

## Manual Build Check

```bash
python -m pip install build twine
python -m build
python -m twine check dist/*
```
