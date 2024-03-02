# Simplexity Semantic Versioning Policy

> [!Note]
> 
> This policy attempts to follow a similar versioning scheme as defined in https://semver.org/ but since we don't really have
> a Public Facing API for most of our stuff, we are gonna just establish what our rules are for versioning for the sake of our own consistency.

## Overview

SemVer specification identifies three main parts of a version that can basically be summed up as...

- Major Version (__X__.y.z): Breaks stuff.
- Minor Version (x.__Y__.z): Adds / Removes stuff.
- Patch Version (x.y.__Z__): Fixes stuff.

This is how we are gonna generally use these.

> [!Important]
>
> The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
> NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and
> "OPTIONAL" in this document are to be interpreted as described in
> [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Minecraft Plugin Development

- Major Version 0 (0.y.z) MUST be used for projects in the development / unstable phase.
  - Generally we will not release these versions.
  - If we release this, then we solved a problem we aimed to solve but we plan on adding more and very likely will be changing a ton of stuff regardless
    or for whatever reason we do not trust our solution.
- Major Versions MUST be incremented when dependencies update that can break previous versions or will require the user to do major changes.
  - Ex: If Minecraft 1.21 adds a new functionality that we use, the major version will be incremented.
  - Ex: If PlaceholderAPI updates and breaks previous implementations of PlaceholderAPI, the major version will be incremented.
  - Ex: If SimplePrefixes decides to merge `prefixes.yml` and `config.yml`, the major version will be incremented.
  - Ex: If SimplePronouns adds a new pronoun type to their configuration, the major version will be incremented.
- Minor Versions MUST reset when the Major Version is incremented.
- Minor Versions MUST be incremented when there is the addition or removal of features that does not break basic server setups.
- Minor Versions SHOULD be incremented if a feature is modified significantly.
- Minor Versions MAY be incremented if an update to the plugin is heavily encouraged due to performance reasons.
  - Ex: If SimplePrefixes adds a new requirement check, the minor version will be incremented.
  - Ex: If a soft dependency is added, the minor version will be incremented.
  - Ex: If SimplePrefixes removes the prefix GUI, the minor version will be incremented.
  - Ex: If a NerfFarms algorithm change reduces the time complexity of an operation from O(n) to O(1), the minor version may be incremented.
- Patch Versions MUST reset when the Minor Version is incremented.
- Patch Versions MUST increment with bug fixes (fixes to internal behavior that resulted in undesired output).
- Patch Versions SHOULD increment with minor changes to features such as simple additional checks.
- Patch Versions SHOULD increment when changes exist that do not fall under the Minor or Major Versions.
  - Ex: If SimplePrefixes adds a simple additional check to an already existing feature, the patch version will be incremented.
  - Ex: If we fix a NullPointerException, the patch version will be incremented.
  - Ex: If we forgot to change a `config.yml` before release, the patch version will be incremented.
- Build Metadata (x.y.z-__META__) will only be included when we use things like Jenkins, which we generally do not use.
- The precedence rules basically match SemVer precedence where the highest Major, highest Minor, then highest Patch are the versions with the highest precedence ([See SemVer Spec 11](https://semver.org/#spec-item-11)).
- If there are many changes in a single version, the change that causes the highest precedence change, will be the version updated.
  - Ex: If a Minor and Patch change is done in the same release, the Minor Version is incremented.
  - Ex: If a Major, Minor, and Patch change is done in the same release, the Major Version is incremented.
