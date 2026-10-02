# Contributing

Thanks for helping! Questions, ideas and bugs are welcome in [Issues](https://github.com/eurojojo/homewizard-dynamic-battery/issues).

## Pull requests

- One topic per pull request, with a short explanation of *why*.
- Keep blueprints configurable: no hard-coded entity IDs, every number an input with a sensible default and a description.
- Shared price logic goes in `custom_templates/home_energy_storage.jinja`, not copied into each blueprint (level 1 is the exception: it must work without extra files).
- Update the matching page in `docs/` and add a line to `CHANGELOG.md` under *Unreleased*.
- `yamllint --strict blueprints packages .github` must pass (the GitHub workflow checks it).
- Test in your own Home Assistant: run **Developer tools → YAML → Check configuration**, and let the automation run for at least a day. Mention in the pull request what you tested, with which devices and price source.

## Versioning

[Semantic versioning](https://semver.org/): a new major version when existing automations need changes (renamed or removed inputs), a minor version for new features, a patch for fixes. Releases are tagged `vX.Y.Z`.
