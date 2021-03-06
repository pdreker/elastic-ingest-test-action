# elastic-ingest-test-action

Tests a set of testcases against an elasticsearch pipeline definition.

You will have to provide your own elasticsearch container by specifying a service in your workflow (see example).

## Limitations

Currently neither supports https, nor authentication. As we are normally running against service containers, this should not be a problem.

## Inputs

### `elastic_version`

The version tag of elasticsearch to use. See <https://hub.docker.com/_/elasticsearch> for available version. Defaults to `7.6.2`. Default may change (within the 7.x.x major version) without notice, so if you need this to be stable, specify your version explicitly.

### `elastic_host`

The hostname where elasticsearch can be reached. If using a service container this is the name of the service specified in your workflow file (also see example). Defaults to `elasticsearch`.

### `elastic_port`

The port where elasticsearch can be reached. Defaults to `9200`

### `pipelines`

**REQUIRED** A space separated list of files declaring elastic pipelines. The paths are relative to your repository and the pipelines in elasticsearch will be named according to the filename after removing path components and the .json suffix - `tests/my-pipeline.json` becomes `my-pipeline`.

### `testdir`

Directory containing the test definitions. All files in this folder are automatically tested against the specified pipeline inside the respective file.

## Outputs

### `testresults`

PASSED or FAILED, depending if the tests passed or failed. Will globally return "failed", if any single test fails. Will return "PIPEFAILED" if deploying pipelines (`--prepare`) failed.

## Example usage

This requires some kind of external elasticsearch. This example uses GitHub's service feature to set htis up. The env vars specify, that we will not be running a multinode cluster (`discovery.type=single-node`) and that we want to use regexes in ingest pipelines (`script.painless.regex.enabled: "true"`). You may have to adapt this to your needs.

Also note that if you give the service a specific name (`my-elastic`) you will have to pass that name as an input (`elastic_host`).

```yaml
jobs:
  check_ingest:
    runs-on: ubuntu-latest
    name: Check Elasticsearch ingest pipelines
    services:
      my-elastic:
        image: elasticsearch:7.5.2
        ports:
          - 9200:9200
        env:
          discovery.type: "single-node"
          script.painless.regex.enabled: "true"
    steps:
      - uses: actions/checkout@v2
      - name: Check elastic pipelines
        id: check_ingest
        uses: pdreker/elastic-ingest-test-action@v1
        with:
          elastic_host: my-elastic
          testdir: 'examples'
```
