# Tuteliq Documentation

Mintlify-powered documentation for the Tuteliq API -- the AI-powered child safety platform.

## The API reference comes from `openapi.json` in this repo

`docs.json` points at the local `openapi.json`, not at a URL.

It used to point at `https://api.tuteliq.ai/docs/json`. Mintlify could not
fetch it: every build logged `Fetched 0 OpenApi file(s)` and then
`Failed to fetch OpenAPI file for anchor or tab`, and the whole site stopped
publishing from late August 2026. The endpoint itself was healthy throughout
(200, valid OpenAPI 3.1, 122 paths), so this was never an API outage, which is
exactly why it went unnoticed for weeks: the failure only appears on the merge
commit, and a pull request shows `Mintlify Deployment: SKIPPED`, which looks
normal.

Fetching a spec from a live URL at build time also couples publishing the docs
to the API being reachable from Mintlify's builders. A vendored file cannot
fail that way.

**Refresh it after any API change that alters the surface:**

```bash
curl -s https://api.tuteliq.ai/docs/json -o openapi.json
```

Check the diff before committing. If `info.version` has not moved and you
expected it to, the deploy you are documenting has not gone out yet.

## Local Development

Preview documentation changes locally:

```bash
npx mint dev
```

Then open [http://localhost:3000](http://localhost:3000) in your browser.

## Deployment

Documentation auto-deploys via Mintlify when changes are pushed to the default branch.

## Links

- **Website:** [tuteliq.ai](https://tuteliq.ai)
- **Documentation:** [docs.tuteliq.ai](https://docs.tuteliq.ai)
