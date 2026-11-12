# Prepare sources and publish content

Run from the deployment root after setup. Keep each plugin manifest authoritative for URLs, branches, and prefixes. Separate workspaces prevent the plugins' identical API-reference paths from colliding.

## Clone sources

```sh
mkdir -p workspaces/words/sources workspaces/tex/sources
git clone --filter=blob:none --sparse --branch master https://github.com/aspose-words-mcp/docs.git workspaces/words/sources/documentation
git -C workspaces/words/sources/documentation sparse-checkout set en/java
git clone --filter=blob:none --sparse --branch main https://github.com/aspose-words-mcp/reference.git workspaces/words/sources/api-reference
git -C workspaces/words/sources/api-reference sparse-checkout set english/java
git clone --filter=blob:none --sparse --branch main https://github.com/aspose-tex-mcp/docs.git workspaces/tex/sources/documentation
git -C workspaces/tex/sources/documentation sparse-checkout set en/java
git clone --filter=blob:none --sparse --branch master https://github.com/aspose-tex-mcp/reference.git workspaces/tex/sources/api-reference
git -C workspaces/tex/sources/api-reference sparse-checkout set english/java
```

Record source commit IDs and keep checkouts clean. These are independent clones, not the engine's legacy source submodules.

## Prepare Gist examples

```sh
PYTHONPATH=plugins/words-java/src plugins/words-java/.venv/bin/python -m adcs_words_java.gist_examples --source-root workspaces/words/sources/documentation/en/java --cache plugins/words-java/sources/gist-examples.json
PYTHONPATH=plugins/tex-java/src plugins/tex-java/.venv/bin/python -m adcs_tex_java.gist_examples --source-root workspaces/tex/sources/documentation/en/java --cache plugins/tex-java/sources/gist-examples.json
```

These explicit network steps pin referenced examples. Extraction uses the caches offline. Preserve cache revisions/checksums with ingestion records and avoid concurrent preparation of the same cache.

## Validate, then publish

```sh
./scripts/ingest.sh words --source-kind both
./scripts/ingest.sh tex --source-kind both
```

Validation may write operation metadata but does not publish. After reviewing results and configuring runtime stores and embeddings, publish each product explicitly:

```sh
./scripts/ingest.sh words --source-kind both --publish
./scripts/ingest.sh tex --source-kind both --publish
```

Run ingestion sequentially. Use `--source-kind docs` or `apiref` to limit a run. `--stage` writes review staging and is separate from `--publish`. Total owns no sources: do not ingest a Total corpus or repeat ingestion for its endpoint.

For durable jobs, add `--enqueue` with an active worker. Read job status with its original product workspace:

```sh
./scripts/run-engine.sh -m adcs.plugins.cli --workspace "$PWD/workspaces/words" --job-status YOUR_OPERATION_ID
```

Use `workspaces/tex` for TeX jobs. Sources, caches, credentials, and generated state are local inputs, not repository content. Verify both products' citations and symbols through Total after publication.
