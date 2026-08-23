# Palomar infrastructure

Last reconciled against the live services on 2026-08-23.

This is the durable record of where Palomar runs, so that changing a host or
credential is a checklist rather than an excavation. The private canonical
database and the public data service are deliberately different systems; never
restore a direct raw-GitHub or GitHub Pages fallback for database data.

## Accounts and ownership

| Thing | Where | Notes |
| --- | --- | --- |
| Repositories | GitHub organization [`PalomarRegistry`](https://github.com/PalomarRegistry) | Moved from the `kim-em` personal account on 2026-08-04. Base member permission is `none`; access is granted by role-specific teams. `PalomarDatabase` and `PalomarSubmissionState` are private. |
| Preserved source | GitHub organization [`PalomarArchive`](https://github.com/PalomarArchive) | Public native forks and immutable record-specific tags. Base member permission is `write`; repository deletion, visibility changes, and private-repository creation are disabled. |
| Archive identity | GitHub user [`PalomarArchivist`](https://github.com/PalomarArchivist) | Dedicated 2FA-protected ordinary member of `PalomarArchive`, never an organization owner or a member of `PalomarRegistry`. |
| Metadata-repair forks | GitHub organization [`PalomarRepairs`](https://github.com/PalomarRepairs) | Created 2026-08-11. Public native forks used only to propose `formalization.yaml` repairs back to submitter-owned repositories. GitHub Actions is disabled throughout the organization, public repository creation is allowed, and private repository creation is disabled. |
| Repair identity | GitHub user [`palomar-repair`](https://github.com/palomar-repair) | Created 2026-08-11. Dedicated 2FA-protected ordinary member of `PalomarRepairs`, never an organization owner or a member of `PalomarRegistry`. |
| Domains | `palomar-registry.org` and `palomarregistry.org`, both at Cloudflare Registrar | Registrar and DNS belong in the dedicated Palomar account. An inter-account Registrar move must be accepted in the dashboard. |
| DNS, Workers, and R2 | Cloudflare account `palomar` (`8e4d5f3bbdd2c4ab2b373721842acf9f`) | All Palomar zones, Workers and buckets belong here. Resource-specific zone ids and nameservers must be recorded after the inter-account transfer creates the destination zones. |
| Website hosting | GitHub Pages, repository `PalomarWeb` | The website is static; its registry content is fetched at runtime from the public data Worker. |
| Public registry storage | Private R2 bucket `palomar-public-data` | Contains the generated, active-only projection: records at keys that never change, and the aggregates under the release that wrote them. The bucket itself is not public. |

The older personal account `d789bf36d237e0cb313be59b927c82bd` is only a
temporary source during the 2026 account separation. It is not an acceptable
deployment target. R2 must first be enabled in the dedicated account; resources
may remain in the old account until the copy, verification, route transfer and
credential rotation are complete.

The old `kim-em/Palomar*` repository names must stay reserved forever.
Recreating a repository at an old name destroys that name's GitHub redirect.
Published records contain immutable URLs, so the cost of losing a redirect only
grows. Redirects are a compatibility convenience, not an archive or recovery
mechanism.

`PalomarRegistry` contains registry infrastructure, not the mathematical work
it evaluates. Submitter-owned repositories and cited sources remain under their
owners' control. A preservation fork in `PalomarArchive` records registered Git
objects without transferring authorship, ownership, or endorsement to Palomar.
Never infer editorial or automation authority from organization membership;
authorize dedicated identities by the specific capabilities their role needs.

The named roles and memberships are maintained in
[`governance.md`](governance.md). Palomar uses GitHub Free and has explicitly
decided not to buy GitHub Team. The organization teams are access groups, not a
paid-plan commitment: Technical Maintainers have Maintain access across the
Palomar repositories, Moderators have only the private Database access needed
for the retraction workflow, and Scientific Advisory Board membership alone
grants no repository access. Public repositories use required checks and
branch protection. The private Database and Submission State cannot receive
the same structural branch protection on the Free plan, so pinned fail-closed
automation, available human review, and manual discipline are the documented
control there.

## Source-preservation organization

`PalomarArchive` is a separate security boundary from the registry repositories.
Its [member privileges](https://github.com/organizations/PalomarArchive/settings/member_privileges)
and [authentication security](https://github.com/organizations/PalomarArchive/settings/security)
must remain configured as follows:

| Setting | Required value | Reason |
| --- | --- | --- |
| Base repository permission | Write | `PalomarArchivist` must add new preservation tags after its temporary per-fork administrator grant is removed. |
| Public repository creation | Allowed | GitHub creates a native organization fork as a new public repository. |
| Private repository creation | Disallowed | Registered source is public and the archive must not become a private-data store. |
| Repository deletion | Disallowed for members | The archive identity cannot erase a preserved fork. |
| Repository visibility changes | Disallowed for members | The archive identity cannot hide or privatize a preserved fork. |
| Require two-factor authentication | Enabled | A member without 2FA cannot retain organization access. |

`PalomarArchivist` is an ordinary active member, not an owner. Creating a native
fork initially gives the creator direct administrator access to that repository.
Registration creates and reads back an immutable-tag ruleset, removes that
direct administrator grant, and only then writes the preservation tag using the
organization's base Write permission. The ruleset allows creation of new
`refs/tags/palomar/**/*` refs but has no bypass actor and rejects updates or
deletions of an existing preservation ref. Organizations cannot star GitHub
repositories, but `PalomarArchivist` is a user account and can. A separate
post-registration reconciler stars only each registered record's original
top-level source—not dependencies or archive forks—and verifies the star before
recording it in private state.

## Metadata-repair organization

`PalomarRepairs` is a separate security boundary for short-lived, public repair
forks. Its
[member privileges](https://github.com/organizations/PalomarRepairs/settings/member_privileges)
and [Actions policy](https://github.com/organizations/PalomarRepairs/settings/actions)
must remain configured as follows:

| Setting | Required value | Reason |
| --- | --- | --- |
| Base repository permission | Read | The organization contains only public forks; broader base write authority is unnecessary. |
| Public repository creation | Allowed | GitHub creates a native organization fork as a new public repository. |
| Private repository creation | Disallowed | Repair inputs and branches are public and the organization must not become a private-data store. |
| GitHub Actions | Disabled for all repositories | A submitter-controlled workflow copied into a fork must never run beside the repair credential. |

[`palomar-repair`](https://github.com/palomar-repair) is an ordinary active
member, not an owner, and has 2FA enabled. Creating a native organization fork
gives it the per-repository authority needed to push and remove the one
deterministic repair branch. The repair worker reads
[`GET /repos/{owner}/{repo}/actions/permissions`](https://docs.github.com/en/rest/actions/permissions#get-github-actions-permissions-for-a-repository)
before every push and proceeds only when `enabled` is `false`; an
organization-policy drift therefore becomes an actionable infrastructure
failure rather than an execution of untrusted workflow code.

The account must never join `PalomarRegistry`, receive access to a private
Palomar repository, or be reused for review, registration, publication, or
source preservation. Organization owners retain the authority to remove stale
repair forks. The repair identity needs no authority to merge its proposals:
the submitter reviews and merges them in the submitter-owned repository.

## Service topology

| Host | Serves | Hosted by | Status |
| --- | --- | --- | --- |
| `palomar-registry.org` | Human-facing website | GitHub Pages, `PalomarWeb` | Live; HTTPS enforced |
| `www.palomar-registry.org` | Redirect to the apex | Cloudflare Redirect Rule | Live; 301; path and query preserved |
| `data.palomar-registry.org` | Active entry records, schemas, renders, evidence, feeds, tombstones, source availability, and the derived pages a reader arrives by. There is no whole-registry document | Cloudflare Worker `palomar-data` over private R2 | Live; read-only |
| `submit.palomar-registry.org` | Submission server | Cloudflare Worker `palomar-server` | Live |
| `palomarregistry.org` and `www.palomarregistry.org` | Defensive-domain redirects | Cloudflare Worker `palomar-domain-redirect` | Live; 308; path and query preserved |
| `palomarregistry.github.io/PalomarWeb/` | Legacy website location | GitHub Pages | Redirects to `palomar-registry.org` |
| `palomarregistry.github.io/PalomarDatabase/` | Nothing | No Pages site | Returns 404 |

The data path is:

```text
private PalomarDatabase main
  -> validate the records the change touched
  -> read the release being served, and stage this one as a difference from it
  -> offer each new record to R2 at a key that never changes
  -> write the aggregates under the new release, its delta last
  -> read every uploaded object back and verify its digest
  -> atomically update R2 _current.json
  -> palomar-data Worker exposes only allowlisted public paths
  -> PalomarWeb fetches https://data.palomar-registry.org/recent.json
```

There is no step there that rewrites a record on the way out, and that is the
change that let the records sit at keys of their own. While the publisher
stripped review scores, a published record's bytes were a function of publisher
code rather than of the commit they came from, so an object called immutable
changed shape twice under a fixed record. The scores live outside the record
now, in the private `scores/`, which staging never opens.

The registered-source path is:

```text
author consents to register a review that identified no blocking problem
  -> PalomarReviewer verifies PALOMAR_ARCHIVE_TOKEN is PalomarArchivist
  -> resolve the submitted repository, every pinned Git dependency, and any
     separately recorded substantive formalization
  -> create or reuse one native PalomarArchive fork per GitHub fork network
  -> install and verify the immutable preservation-tag ruleset
  -> remove PalomarArchivist's direct repository administrator grant
  -> create and read back a record-specific tag for every registered commit
  -> bind source-archive.json and the preservation map into the database record
  -> only then publish the PalomarDatabase registration branch
```

Any failure in that path stops registration before a database branch is
published. A source shape that a native fork cannot preserve is rejected during
mechanical verification rather than discovered during registration.

The metadata-repair path is separate from review and registration:

```text
failed verification publishes structured, schema-backed repairable fields
  -> the author chooses whether to request a repair
  -> PalomarSubmissionState records the request in index/repairs.json
  -> repairer.yml validates the proposed values through the shared preflight
     implementation used by submissions
  -> palomar-repair creates or reuses an Actions-disabled PalomarRepairs fork
  -> write one deterministic repair branch and read it back
  -> open a pull request to the submitter-owned repository
  -> the author reviews and chooses whether to merge it
```

A repair pull request is a proposal, not a registered record or a preserved
source. It never grants `palomar-repair` access to `PalomarRegistry`, and the
repair worker refuses to push unless GitHub reports that Actions is disabled on
the destination fork.

The Worker never exposes `_current.json`, a release delta, either internal key
prefix, bucket listings, the private `takedowns.json`, or submitter identity. A
request carrying a query string is refused with 404 before anything is read,
because Cloudflare's cache key includes one and nothing served here takes a
parameter. Two things this list used to name are no longer withheld so much as
absent. `index.json` is a staging intermediate that reaches R2 under no key at
all, having been the one served object whose size was the registry's. Review
scores are not a field held back from the projection either; they are not in the
record, and `review` is `additionalProperties: false`, so a record carrying them
fails the schema check staging already runs rather than relying on a stripping
step that could be forgotten. The Worker has an R2 binding named `DATA` and no
secrets. A missing or invalid current-release pointer fails closed with 503;
there is no raw-GitHub fallback.

`palomarregistry.org`, without the hyphen, is a defensive registration. It is a
separate Cloudflare zone and serves no content of its own.

## Cloudflare resources

| Resource | Configuration source | Deployment |
| --- | --- | --- |
| Worker `palomar-data-staging` | `PalomarDatabase/worker/wrangler.jsonc`, top level | Manual staging deployment at its `workers.dev` URL |
| Worker `palomar-data` | `PalomarDatabase/worker/wrangler.jsonc`, environment `production` | Manual deployment with `npx wrangler deploy --env production`; owns the `data` custom domain |
| R2 bucket `palomar-public-data` | `PalomarDatabase` publication tooling and the Workers `DATA` binding | Snapshots are published by private GitHub Actions, not by Worker deployment |
| Worker `palomar-server` | `PalomarServer/wrangler.jsonc` | Tested, uploaded, and promoted automatically on pushes to `main` |
| Worker `palomar-domain-redirect` | `PalomarServer/redirect/wrangler.jsonc` | Manual `npm run deploy:redirect` |
| `www.palomar-registry.org` redirect | Cloudflare zone Redirect Rule | Managed separately from Wrangler |

Deploying `palomar-data` changes the reader, not the data. Publishing a database
snapshot changes R2, not the Worker. Keep those operations separate so either
can be rolled back without changing the other.

## DNS

| Name | Configuration | Proxy/owner |
| --- | --- | --- |
| `palomar-registry.org` | GitHub Pages A records `185.199.108.153` through `185.199.111.153` and the corresponding AAAA set | DNS only; GitHub terminates TLS |
| `www.palomar-registry.org` | Placeholder record used only to enter Cloudflare's proxy | Proxied; Redirect Rule answers at the edge |
| `data.palomar-registry.org` | Worker custom domain declared by `PalomarDatabase/worker/wrangler.jsonc` | Managed by Cloudflare/Wrangler; do not recreate the retired GitHub Pages CNAME |
| `submit.palomar-registry.org` | Proxied record plus the Worker route in `PalomarServer/wrangler.jsonc` | Cloudflare Worker route |
| `palomarregistry.org`, `www.palomarregistry.org` | Worker custom domains declared by `PalomarServer/redirect/wrangler.jsonc` | Managed by Cloudflare/Wrangler |
| `_github-pages-challenge-palomarregistry` | GitHub organization Pages verification token | DNS only |

The apex remains grey-clouded because GitHub must receive the request and
terminate TLS. `data` must not point at `palomarregistry.github.io`: the old
CNAME was deleted before the Worker custom domain was deployed. Wrangler owns
the data custom-domain record, so changing it by hand risks disconnecting the
Worker.

## Render-origin isolation

Render bundles are untrusted output derived from submitter-controlled Lean
source. They are served from `data.palomar-registry.org`, while the parent site
is served from `palomar-registry.org`. PalomarWeb pins that render base and its
`frame-src` CSP to the data host. Embedded documents also use
`sandbox="allow-scripts"` without `allow-same-origin`, giving the frame an opaque
origin.

The distinct hostname, iframe sandbox, exact render CSP, database content
validator, trusted runtime hashes, and browser sanitizer are independent
layers. Do not remove one because the others exist. The data host must not set
application cookies or host the website, and render paths must remain immutable
and content-addressed.

## Certificates and HTTPS

GitHub Pages terminates TLS for `palomar-registry.org`; Cloudflare terminates TLS
for `www`, `data`, `submit`, and the defensive-domain redirects. Certificates
are managed by those providers and are not installed manually.

HTTPS enforcement for the website is the `https_enforced` setting on the
`PalomarWeb` Pages site. It does not apply to the data Worker. When testing a
Pages enforcement change, request an uncached path because Pages responses may
remain cached for several minutes:

```sh
curl -sSI "http://palomar-registry.org/nonexistent-$RANDOM.html" # expect 301
```

The hyphenated `www` Redirect Rule is:

```text
expression:  http.host eq "www.palomar-registry.org"
target:      concat("https://palomar-registry.org", http.request.uri.path)
status:      301, preserve_query_string: true
```

Create a redirect rule before proxying a redirect-only name, and use a dynamic
target so deep links and query strings survive. The defensive-domain Worker
performs the equivalent redirect with status 308.

## Credentials

Never record credential values or filesystem locations here.

| Credential | Scope and use | Held by |
| --- | --- | --- |
| Wrangler OAuth login | Worker scripts, routes, bindings, and read-only account inspection used for manual deployments | `kim@lean-fro.org` |
| DNS/Redirect API token | DNS edit, zone read, and Single Redirect edit on the two Palomar zones only | `kim@lean-fro.org` |
| R2 publisher Account API token | Object read/write/list on the single `palomar-public-data` bucket; no bucket administration, Worker deployment, DNS, registrar, or billing access | GitHub Actions in private `PalomarDatabase` |
| Server deployment API token | Worker version upload and promotion for `palomar-server` | GitHub Actions in `PalomarServer` |
| Registry automation token (`PALOMAR_GITHUB_TOKEN`) | Reads verification artifacts, updates private submission state, and creates and merges registration changes in the private database | GitHub Actions in private `PalomarSubmissionState` |
| Archive token (`PALOMAR_ARCHIVE_TOKEN`) | Authenticates only as `PalomarArchivist`; creates and writes native public forks in `PalomarArchive`, manages each new fork's preservation ruleset and direct creator grant, and stars original sources after registration. A classic token needs `public_repo` and `workflow`; GitHub requires `workflow` even for a tag whose commit contains a workflow file. | GitHub Actions in private `PalomarSubmissionState` |
| Repair token (`PALOMAR_REPAIR_TOKEN`) | Authenticates only as `palomar-repair`; creates public forks in `PalomarRepairs`, verifies their Actions policy, pushes and removes one deterministic repair branch, and opens pull requests to submitter-owned public repositories. It is a classic token with only the top-level `repo` scope: GitHub requires that scope to read a repository's Actions-permissions endpoint, while account isolation prevents access to Palomar's private repositories. | GitHub Actions in private `PalomarSubmissionState` |
| Review-engine API key (`OPENAI_API_KEY`) | Runs the private editorial review pipeline | GitHub Actions in private `PalomarSubmissionState` |

The private Database repository stores only these R2 publisher secrets:

- `CLOUDFLARE_ACCOUNT_ID`
- `R2_ACCESS_KEY_ID`
- `R2_SECRET_ACCESS_KEY`

For an R2 Account API token, select the `R2 buckets` resource scope and restrict
it to `palomar-public-data`. The required bucket permission is
`Workers R2 Storage Bucket Item Write`, which permits reading, writing, and
listing objects without bucket administration. Cloudflare's S3 mapping is:

- Access Key ID: the API token ID;
- Secret Access Key: the SHA-256 digest of the API token value.

The token-creation confirmation may show those two S3 values directly. Copy
them then: the secret is not shown again. Install the mapped values as the two
`R2_*` GitHub Actions secrets, not as Worker secrets and not in repository
variables. Rotating the parent token requires replacing both mapped secrets.

The `palomar-data` Worker itself needs no credential because its R2 access is a
binding. The publisher needs S3 credentials because it runs in GitHub Actions.

The private SubmissionState repository stores these reviewer secrets:

- `OPENAI_API_KEY`;
- `PALOMAR_GITHUB_TOKEN`;
- `PALOMAR_ARCHIVE_TOKEN`;
- `PALOMAR_REPAIR_TOKEN`.

The exact repository permissions for `PALOMAR_GITHUB_TOKEN`, archive-account
guardrails, and rotation procedure are maintained in the
[`PalomarSubmissionState` runbook](https://github.com/PalomarRegistry/PalomarSubmissionState#secrets).
Keep the combined archive credential as a classic token with `public_repo` and
`workflow`. A fine-grained token scoped to `PalomarArchive` could preserve its
forks with Metadata read plus Administration, Contents, and Workflows write,
but it cannot also star arbitrary public source repositories owned by unrelated
accounts. Splitting preservation and starring across credentials would require
an explicit workflow and secret-design change. GitHub documents the separate
[Workflows repository permission](https://docs.github.com/en/rest/authentication/permissions-required-for-fine-grained-personal-access-tokens#repository-permissions-for-workflows)
and [Starring user permission](https://docs.github.com/en/rest/activity/starring#star-a-repository-for-the-authenticated-user).
Do not combine the registry and archive credentials: the archive identity must
not be able to mutate `PalomarRegistry`, and the general registry automation
identity must not bypass the archive's dedicated-account check.

Do not combine the repair credential with either of those identities. The
classic `repo` scope is broader than the repairer's public-only work would
otherwise suggest because GitHub's Actions-permissions endpoint requires it.
That scope is contained by keeping `palomar-repair` out of every private
repository and organization. The account must not acquire a personal private
repository while this credential exists.

## Deployment and recovery

### Website

Pushes to `PalomarWeb/main` test, build, and deploy GitHub Pages. The hourly
`Published site health` workflow checks that the live site names the current
commit and that the website's own validators can load `recent.json` and every
record it names. It asked about every active entry while a document named them
all; there is no such document now, and the page a visitor actually loads is
both the check that matters and the only one whose cost does not grow with the
registry. It dispatches one fresh Pages deployment when the served build is
stale and fails loudly when the public projection has become incompatible with
the website contract.

### Submission server

Pushes to `PalomarServer/main` run tests, upload a Cloudflare Worker version,
and promote it. The workflow does not change its route or cron trigger.

One piece of operator state has no dashboard and appears in no repository the
public can read: `index/rate/<digest>.json` in `PalomarSubmissionState`, the
interval a submitter must wait before starting another submission. It is sixty
seconds to begin with and doubles on every start, and only a completed
registration puts it back to the floor; a failed verification or a withdrawal
leaves it where it is, because those are the loops worth slowing down. There is
deliberately no ceiling, so somebody who has locked themselves out is released
by an operator deleting that one file, and the file records the login and the
time so an operator can tell whose it is. Its name is a peppered digest of the
principal rather than a login, so listing the directory does not enumerate who
has submitted.

### Review, registration, source preservation, and metadata repair

The private `PalomarSubmissionState` workflow runs every two hours, at `37 */2`,
and may also be dispatched manually. A pass that has more to do asks for the
next one rather than waiting for the clock, so the interval sets how long an
idle registry sleeps and not how fast a busy one moves. Both production paths
that execute Reviewer—the ordinary `reviewer.yml` pass and the weekly
`queue-sweep.yml` rebuild—install it from a full commit SHA, never mutable
`main`, and use the same bounded dependency cutoff. State's least-authority CI
installs that same commit and exercises its command surface without receiving
secrets or write authority. State's contract tests require all three installs,
dependency cutoffs, and setup action pins to agree. The daily and manually
dispatchable `pin-currency.yml` then compares the production commit with
Reviewer `main` and fails once it is more than one commit behind; advancing
production remains an explicit reviewed State change rather than an automatic
update.

The ordinary pass runs `palomar-review doctor` and then advances each live
submission. Most transitions are one step per pass; a registration is not, and
runs to completion inside the pass that opened it, so that opening the database
change, waiting for its validation, merging, and finalizing are not four passes
and up to eight hours apart. The finalize arm is recovery for a registration
whose job died between opening the change and merging it. Review, registration,
and finalization are serialized by one workflow concurrency group.

That pass reads its work from `index/open.json`, an index of the open
submissions that each transition maintains as it goes. Listing `submissions/`
through the contents API instead meant a request whose cost was the number of
submissions the registry had ever had, paid on every scheduled pass forever.
A weekly `queue-sweep.yml` rebuilds the index from the state directories, so an
index that has drifted is repaired rather than believed indefinitely.

After advancing submissions, the same workflow runs `palomar-review
star-registered`. It uses GitHub's
[`PUT /user/starred/{owner}/{repo}` endpoint](https://docs.github.com/en/rest/activity/starring#star-a-repository-for-the-authenticated-user),
reads the star back, and only then records the account, repository, and time in
private submission state. A failed call cannot undo or block a registered record;
because no success is recorded, the next scheduled pass retries it.

After creating or rotating `PALOMAR_ARCHIVE_TOKEN`, sign in as the archive
account at <https://github.com/settings/tokens>. Editing an existing classic
token's scopes preserves its value; a replacement token must also replace the Actions secret at
<https://github.com/PalomarRegistry/PalomarSubmissionState/settings/secrets/actions>,
and run a preflight with new model reviews disabled:

```sh
gh workflow run reviewer.yml \
  --repo PalomarRegistry/PalomarSubmissionState \
  --ref main -f max_reviews=0
```

The `Check prerequisites` log must say
`archive token: PalomarArchivist (verified)`. This proves the installed secret's
identity and organization access without creating a synthetic test fork.
`max_reviews=0` suppresses new model reviews but does not suppress registration
or finalization of an already-ready submission; inspect the private inflight
state first if the run must be credential-only. The first real registration
additionally proves fork creation, ruleset creation, administrator demotion, tag
creation, and read-back; it fails closed before publication if any one of those
operations is unavailable.

The separate `repairer.yml` workflow runs every two hours, at `17 */2`, and may
also be dispatched manually. It reads `index/repairs.json`, installs
`PalomarReviewer` and `PalomarSubmission` from their public `main` branches,
and runs the repair proposal through the same preflight implementation used by
submitters. Its concurrency group serializes repair branches and pull-request
creation independently of editorial review and registration.

After creating or rotating `PALOMAR_REPAIR_TOKEN`, sign in as
[`palomar-repair`](https://github.com/palomar-repair) and create a classic token
at <https://github.com/settings/tokens> with only the top-level `repo` scope.
Record its expiration in the operator credential manager. Replace the Actions
secret at
<https://github.com/PalomarRegistry/PalomarSubmissionState/settings/secrets/actions>,
then revoke the superseded token and dispatch the workflow:

```sh
gh workflow run repairer.yml \
  --repo PalomarRegistry/PalomarSubmissionState \
  --ref main
```

An empty repair queue proves the production workflow and installed command can
start with the secret configured; it does not exercise the token against a
fork. The next real repair is the end-to-end credential check: it verifies the
account can create or reuse the organization fork, reads back that Actions is
disabled before pushing, reads back the branch, opens the pull request, and
records the result in private State. Any failure leaves the submission and its
repository unchanged and records an actionable infrastructure failure for an
operator. To retire the repair facility, disable `repairer.yml`, remove
`PALOMAR_REPAIR_TOKEN`, and revoke the token before removing the account from
`PalomarRepairs`.

For a newly registered record, inspect its `preservation.repositories` mapping
and verify each linked [PalomarArchive repository](https://github.com/PalomarArchive),
the repository ruleset under **Settings → Rules → Rulesets**, and the exact
record-specific tag. The six-hour source-availability workflow then checks both
the original and preserved commits, and fails visibly if a preserved copy is
confirmed missing twice consecutively. What it knew last time it reads back from
the object it is serving, and it writes the active-only manifest straight to that
same key. The complete state used to sit on a private `availability` branch, and
that branch is gone: keeping it meant a workflow with write access to the
canonical repository, a commit every six hours, and two copies of the truth that
could disagree. The workflow now runs with `contents: read`.

### Public registry data

Pushes affecting publishable Database inputs first read the content-addressed
release delta currently served from R2, through authenticated bucket access.
Its database commit is the exact base for both validation and staging, so a
publication that is catching up after a failed run validates every accumulated
change it will publish. An absent or unreadable delta, or a commit that cannot
be resolved as a current ancestor, selects complete validation and staging; a
document that disagrees with its content-addressed release identifier is
refused outright.

The release then uploads what changed, reads it back, verifies its digests, and
updates `_current.json` last. Publication alternates credentialed steps with
uncredentialed ones on purpose: staging has no bucket credentials and must keep
none, so every step that decides what a release contains has no access, and
every step that has access decides nothing. The plan is worked out twice because
a word's open postings page cannot be named until that word's head has been
read.

A successful publication uploads its canonical release delta and the entry
records its health check needs—the versions that release added and every
version currently taken down—as an Actions artifact bound to that run and
attempt. `Published evidence health` downloads it through the authenticated
Actions API, verifies that the delta names the triggering commit, and checks
only the versions that release added while retaining the complete withdrawal
check. This is the ordinary event path; it does not reopen every historical
render and evidence bundle.

Complete published-evidence work is a different path. A daily run, a manual
dispatch, a failed publication, missing or malformed run-bound evidence, a
commit mismatch, or a failed delta check uses a fresh checkout of current
Database `main`, checks every registered render, and audits the complete served
R2 release. A missing object can dispatch one filtered-data recovery
publication. This is a fail-closed whole-origin evidence audit, not the full
canonical-database reconstruction.

The weekly and manually dispatchable `whole-database-sweep.yml` owns that final
layer: it walks first-parent append-only history, checks every frozen path's
mode, validates every record from its bytes, stages a full public dataset, and
reconciles every served page against that rebuild. Keeping this reconstruction
weekly preserves the independent whole-database check without charging it to
each registration or conflating it with the daily published-evidence
audit.

Before changing the data Worker, deploy and check `palomar-data-staging`. Then
deploy production and verify at least:

```sh
curl --fail https://data.palomar-registry.org/healthz
curl --fail https://data.palomar-registry.org/recent.json
curl --fail --head https://data.palomar-registry.org/recent.json
curl -X POST -o /dev/null -w '%{http_code}\n' https://data.palomar-registry.org/recent.json # expect 405
curl -o /dev/null -w '%{http_code}\n' 'https://data.palomar-registry.org/recent.json?v=1' # expect 404
curl -o /dev/null -w '%{http_code}\n' https://data.palomar-registry.org/index.json # expect 404
curl -o /dev/null -w '%{http_code}\n' https://data.palomar-registry.org/_current.json # expect 404
```

The two 404s are the point of the check, not an afterthought. `index.json` was
the one served object whose size was the registry's, and it is gone; a query
string is refused before any read, because Cloudflare's cache key includes one
and ignoring it would let any client mint unbounded misses against the bucket.

The obsolete Database Pages site should remain absent. Anonymous GitHub API,
raw-content, and former Pages requests for `PalomarDatabase` should all return
404, while authenticated maintainers retain Git and API access.

## Moving a hostname

Hostnames are pinned in code, CSPs, feeds, workflows, and immutable records.
For a data- or website-domain change, audit at least:

| Where | What |
| --- | --- |
| `PalomarWeb/assets/security.mjs` | `DEFAULT_DATABASE`, `DEFAULT_RENDER_BASE`, source-availability URL, and every allowlisted read-surface URL builder |
| `PalomarWeb/assets/app.js` | canonical website and feed bases |
| `PalomarWeb/{index,entry,render,404,about}.html` | CSP `connect-src`/`frame-src`, feed links, and data links |
| `PalomarWeb/.github/workflows/*.yml` and tests | pinned public-data and website origins |
| `PalomarDatabase/worker/wrangler.jsonc` | Worker custom domain |
| `PalomarDatabase/tools/{build_feeds.py,check_published.py}` and `tests/` | website, feed, and public-data bases |
| `PalomarDatabase/docs/*.md` | publication and render-origin runbooks |
| `PalomarReviewer/src/palomar_reviewer/cli.py` | public website/data URLs used during registration |
| `PalomarServer/wrangler.jsonc` | website URL and submission route |
| `PalomarServer/src/index.js` and `public/{intake.js,llms.txt}` | CSP `connect-src`, the versions endpoint the status page reads, and the hosts the agent instructions name |
| `PalomarServer/redirect/index.js` | `CANONICAL_ORIGIN` for the defensive domains |

Registered entries, evidence, renders, and in-use schemas are immutable. Never
rewrite their historical URLs. Keep an old hostname serving or redirecting as
long as an immutable record names it. Feed XML regenerates, but subscribers may
retain old copies.

For a new Pages website domain: verify it at the GitHub organization, create
the complete DNS set, attach it to `PalomarWeb`, wait for the certificate, then
enforce HTTPS. For a new data domain: add it to the production Worker config,
deploy and verify the Worker, update all consumers and CSPs, deploy them, and
only then retire the old Worker custom domain. Never point either data hostname
at raw GitHub content.

## What is deliberately manual

Some operational state exists only in provider control planes and will not
appear in a repository search: organization membership and permissions,
repository visibility, branch protection and rulesets, Pages custom domains and
HTTPS settings, Actions secrets and variables, installed Apps and webhooks,
issue labels, Cloudflare account resources, and the reservation of old
repository names. Audit this inventory explicitly during an ownership,
organization, or hostname migration.

Registrar operations, payment, organization-level GitHub Pages domain
verification, the `www` Redirect Rule, data-Worker staging/production
deployments, creation and recovery of the `PalomarArchivist` GitHub account,
creation and recovery of the `palomar-repair` GitHub account, archive- and
repair-organization policy changes, and credential rotation are manual.
Website, server, review, registration, source preservation, metadata repair,
filtered-data, availability, health-check, and the two weekly sweep workflows
are automated.
The sweeps are where anything whose cost is the size of the registry rather than
the size of the change has been moved, because those checks used to run once per
registered result and the registry paid for them quadratically over its life. The
account is
currently using the Cloudflare Workers and R2 free tiers; usage and paid-plan
triggers are tracked in the workspace `TODO.md`.
