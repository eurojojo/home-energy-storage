# Contributing

Thanks for wanting to help. Questions, ideas and bug reports all go in [Issues](https://github.com/eurojojo/home-energy-storage/issues).

## Pull requests

Keep each pull request to one topic, and say briefly why you made the change.

Blueprints should stay configurable. Don't hard-code entity IDs, and make every number an input with a sensible default and a short description. Price logic that more than one blueprint needs belongs in `custom_templates/home_energy_storage.jinja`, not in a copy inside each blueprint. Level 1 is the exception, because it has to work without extra files.

Update the matching page in `docs/` and add a line to `CHANGELOG.md` under *Unreleased*. The GitHub workflow runs `yamllint --strict blueprints packages .github`, so run that yourself before you push.

Test in your own Home Assistant. Run the configuration check in Developer tools → YAML and let the automation run for at least a day. In the pull request, mention what you tested, with which devices and which price source.

## Versions

The version numbers follow [semantic versioning](https://semver.org/). A change that forces people to edit their automations (an input renamed or removed) gets a new major version. New features bump the minor version and fixes the patch version. Releases are tagged `vX.Y.Z`.
