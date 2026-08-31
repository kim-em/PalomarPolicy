# Maintainer guide: exceptional registry corrections

This guide is for Palomar Technical Maintainers who need to correct descriptive
metadata on an already registered entry. Registry corrections are exceptional
housekeeping. They are not a way to change or re-run a formalization, and they
do not express Palomar's approval or endorsement of it.

## Before you start

Use a Registry correction only when public metadata is wrong or materially
misleading and it is not reasonable to wait for the project to register a new
version. If the repository, source commit, project path, `formalization.yaml`
path, or Comparator configuration path must change, stop: that requires an
ordinary new version submitted by the project, not a Registry correction.

Have these ready:

- the entry's `PALOMAR-...` identifier;
- the corrected values, checked against an authoritative source; and
- a short public explanation of what was corrected and why.

The explanation, changed-field list, and attribution `Palomar / Registry
correction` will be permanently visible in the entry's version history. Write
the explanation for readers outside the maintainer team. The individual
operator's GitHub identity remains in private operational state.

## Create the correction

1. Open the [Palomar Technical Maintainer dashboard](https://submit.palomar-registry.org/dashboard).
2. Sign in to GitHub with an account whose numeric ID is in Palomar's current
   Technical Maintainer allowlist. The dashboard session lasts 15 minutes.
3. Under **Registry maintenance**, select **Create an exceptional metadata
   correction**.
4. Enter the existing Palomar identifier. The editor loads the highest active
   version; verify that the displayed version is the one you intend to correct.
5. Edit only the inaccurate descriptive fields. The editor permits title,
   abstract, authors, arXiv and MSC 2020 classifications, responsible
   maintainers, mathematical sources, and related formalizations.
6. Enter the public explanation. State the incorrect value or omission, the
   corrected information, and the basis for making the change. Do not include
   private correspondence, credentials, or personal data that should not be
   published.
7. Select **Validate correction**. Complete the GitHub identity check if asked.
   Palomar checks Technical Maintainer membership again before admitting the
   correction.
8. Save the resulting private status URL. Follow it until the metadata review
   and registration finish, and address any reported validation problem there.

Palomar computes the changed-field list rather than trusting the browser. It
also rejects a stale correction if another version became active after the
editor loaded, and rejects any change to the repository, commit, or registered
paths.

## After registration

Open the public entry and inspect its version history. Confirm that:

- a new version appears with the highlighted **Registry correction** label;
- expanding the label shows the intended explanation and changed fields;
- the repository, commit, project path, metadata path, and Comparator path are
  identical to the baseline version; and
- the corrected metadata is rendered as intended.

Do not edit an earlier database entry to repair a mistake. If the new
correction itself is wrong, append another correction. If a published version
must be hidden for a policy or safety reason, use the separate takedown process;
registration history remains append-only.

## Troubleshooting

- **The dashboard refuses sign-in:** confirm that you used the GitHub account
  recorded in the Technical Maintainer allowlist. Ask another maintainer to
  review the allowlist if the account changed.
- **The editor says the correction is stale:** reload the entry in the editor
  and redo the correction against the newly active version.
- **A repository, SHA, or path needs changing:** use the ordinary project
  submission process. This correction mechanism deliberately cannot do that.
- **The dashboard session expires while editing:** reopen the dashboard, sign
  in again, and reload the entry. Keep a copy of a long explanation before
  refreshing.
- **Validation or review fails:** use the private status page's findings. Do not
  work around the correction contract by editing generated database files.

The normative policy is
[section 9.1 of `CONTRIBUTING.md`](../CONTRIBUTING.md#91-exceptional-registry-metadata-corrections).

