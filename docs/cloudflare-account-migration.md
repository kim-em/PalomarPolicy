# Cloudflare account-separation runbook

This runbook moves project resources out of the personal Cloudflare account
`d789bf36d237e0cb313be59b927c82bd`. The final ownership map is:

| Project | Destination account |
| --- | --- |
| Hex | `5acf032f740d48aa656788e28cabcf2e` |
| Palomar | `8e4d5f3bbdd2c4ab2b373721842acf9f` |
| TauCeti | `ec2169bdf033f56b009956d4b64ba8ef` |

The personal account is empty at completion. `lean-eval`
(`a46b90978a1c29cc4795f30677e7e4b8`) is already correctly separated and is a
verification-only target for this migration.

## Preconditions

1. Enable R2 billing in the Hex, Palomar and TauCeti dashboards. Wrangler
   reports API error 10042 until this has been done once per account.
2. Put the temporary bootstrap API token in a local mode-0600 file. It needs
   account-scoped Workers and R2 edit access on the source and three
   destinations, and zone/DNS/rules edit access where Cloudflare offers those
   permissions. Never commit it or pass it on a command line.
3. Create the replacement GitHub OAuth client secret and the least-privilege
   GitHub runtime token. Keep both in mode-0600 local files until they have been
   installed as Worker secrets.
4. Generate a new high-entropy Palomar `TOKEN_PEPPER`, store it in the password
   manager, and expose it to the rekey command only through its environment.
5. Merge the account-targeting PRs. Set `CLOUDFLARE_ACCOUNT_ID` as a GitHub
   variable, not a secret, in PalomarServer, PalomarDatabaseTools' `production`
   environment and PalomarDatabase. Do not rotate the R2 publisher/cache keys
   yet.

Record pre-migration object counts and sizes, Worker version ids, routes, DNS,
email routes, redirect/cache rules and bucket public-host settings. Export the
zone DNS records before initiating a Registrar move.

## Copy before cutover

Create destination buckets in the same locations as their source buckets:

- `hex-cache` in ENAM;
- `palomar-public-data` and `palomar-public-data-staging` in ENAM; and
- `tauceti-cache` in WEUR.

Copy all objects with an R2-compatible client using separately scoped source
read and destination write credentials. A copy is accepted only when recursive
listings agree on key, size and checksum where R2 supplies one, aggregate byte
and object counts agree, and representative artifacts can be downloaded and
used. Run a final incremental copy immediately before each cache endpoint is
changed.

Deploy the destination Workers without traffic first. Install freshly created
secrets; do not copy a credential whose authority is tied to the personal
account. Verify that alternate `workers.dev` and preview hostnames remain off.

## Cache cutovers

For Hex and TauCeti, stop cache publication, make the final incremental copy,
then replace the GitHub Actions upload endpoint and key together. Hex also gets
the destination bucket's newly allocated `r2.dev` hostname. TauCeti keeps
`cache.taucetiproject.org`, attaching it to the new bucket only after the zone
has moved. Trigger one trusted main-branch cache publication and one
unprivileged read build before deleting the source bucket.

## Palomar cutover

1. Deploy PalomarServer with `PALOMAR_WRITES_PAUSED` set to `true`. Confirm
   `/healthz`, status pages and the dashboard still read, while every mutation
   and scheduled reconciliation returns without writing.
2. Run the rekey tool from the reviewed PalomarServer checkout against a fresh
   PalomarSubmissionState checkout. First run without `--write`; check its
   record, principal and token counts. Run again with `--write`, validate State,
   inspect the diff and merge the private State PR.

   ```console
   TOKEN_PEPPER="$TOKEN_PEPPER" tools/rekey-pepper-indexes.js ../PalomarSubmissionState
   TOKEN_PEPPER="$TOKEN_PEPPER" tools/rekey-pepper-indexes.js --write ../PalomarSubmissionState
   ```

   Principal indexes are regenerated and rate indexes are reset. Historical
   token pointers remain because State validation binds them to the digests in
   each record; the new pepper nevertheless makes the old raw links unusable.
3. Make and verify the final public-data bucket copy. Install the new pepper,
   OAuth secret and GitHub runtime token in the destination Worker.
4. In Cloudflare Registrar, click **Start move** for
   `palomar-registry.org`, `palomarregistry.org` and `taucetiproject.org`; then
   click **Accept** in each destination account. These six dashboard actions
   cannot be performed by Wrangler or the API. Reconcile every new zone's DNS,
   Email Routing, redirect/cache rules and SSL settings against the export.
5. Attach Worker routes and custom domains in the destination accounts. Switch
   Palomar R2 publisher credentials and the public-data Worker binding. Confirm
   the website, data endpoint, submit endpoint, defensive redirects, mail route
   and TauCeti cache domain.
6. Deploy PalomarServer with `PALOMAR_WRITES_PAUSED` set to `false`. Recover at
   least one pre-migration open submission through GitHub identity, submit a
   controlled test and verify the scheduled handler completes.

## Acceptance and deletion

Before deletion, compare live behavior with the recorded inventory and inspect
Cloudflare logs for errors. Confirm the destination object inventories again,
all GitHub repository variables and secrets target the destination accounts,
and no workflow still references the personal account id.

Delete source Workers and buckets only after those checks pass. The approved
policy is immediate source deletion after acceptance, with no retention copy.
Revoke the bootstrap token and every superseded R2, Worker, GitHub OAuth and
GitHub runtime credential. Finally survey all five Cloudflare accounts: the
personal account must contain no project resource, each project account must
contain only its own resources, and lean-eval must be unchanged.
