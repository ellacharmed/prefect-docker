[![View on Medium](https://img.shields.io/badge/Medium-View%20on%20Medium-red?logo=medium)](https://towardsdatascience.com/create-robust-data-pipelines-with-prefect-docker-and-github-12b231ca6ed2?sk=56087dca06789be3b018c884d6a90f02)

# prefect-docker

Demo on how to use [Prefect with Docker (upstream repo by khuyentran1401)](https://github.com/khuyentran1401/prefect-docker)

## Changes from upstream

Copied text below from linked medium article, to have all information in one place.

commands:

### Build a Deployment

```python
prefect deployment build src/main.py:create_pytrends_report \
  -n google-trends-gh-docker \
  -q test
```

where:

- `-n google-trends-gh-docker` specifies the name of the deployment to be `google-trends-gh-docker`.
- `-q test` specifies the work queue to be `test`. A work queue organizes deployments into queues for execution.

### Create a Deployment via the API

```python
prefect deployment apply create_pytrends_report-deployment.yaml
```

Above 2 commands can be merged. To combine the `prefect deployment build` and `prefect deployment apply` steps into one step, add the `--apply` option to `prefect deployment build `

```python
prefect deployment build src/main.py:create_pytrends_report \
  -n google-trends-gh-docker \
  -q test \
  --apply
```

### Instantiate a Docker Container Block

1. Blocks menu -> Docker Container -> Add
1. Fill in fields as shown:
    - **Block name**: Name of the block
      - `google-trends`
    - **Env**: Environment variables to set in the configured infrastructure. We can use `EXTRA_PIP_PACKAGES` to install dependencies at run time
      ```
      {
        "EXTRA_PIP_PACKAGES" : "datapane==0.15.1 plotly==5.10.0 prefect>=2.3.2 prefect_shell==0.1.1 pytrends==4.8.0" 
      }
      ```
    - **Type (Optional)**: Type of infrastructure
      - `docker-container`
    - **Image (Optional)**: Tag for the docker image
      - ~~`ellacharmed/google-trends:first`~~ (use default Prefect's Docker image)
    - **Stream Output**: If set, the output will be streamed from the container to the local standard output
      - ON

### Instantiate a GitHub Storage Block

1. Blocks menu -> Github -> Add
1. Fill in fields as shown:
    - **Block name**: Name of the block
      - `pytrends`
    - **Reference (Optional)**: Branch name or tag
      - `master`
    - **Repository**: URL to GitHub repo (https/ssh)
      - `https://github.com/ellacharmed/prefect-docker`

### Create a Deployment with Docker Infrastructure + GitHub Storage

Create a deployment with the two blocks we have just created

```python
prefect deployment build src/main.py:create_pytrends_report \
  -n google-trends-gh-docker \
  -q test \
  -sb github/pytrends \
  -ib docker-container/google-trends \
  -o prefect-docker-deployment \
  --apply
```
where:

- `-n google-trends-gh-docker` specifies the name of the deployment to be `google-trends-gh-docker`
- `-q test` specifies the work queue to be `test`
- `-sb github/pytrends` specifies the storage to be the - `github/pytrends` block
- `-ib docker-container/google-trends` specifies the infrastructure to be the `docker-container/google-trends` block
- `-o prefect-docker-deployment` specifies the name of the YAML file to be `prefect-docker-deployment.yaml`

### Run a Deployment

To execute flow runs from this deployment, start an agent that pulls work from the pool created in previous lesson:

1. Start Docker Desktop
1. `- prefect agent start -p 'zoompool'`
1. Deployment menu - look for `google-trends-gh-docker` deployment -> drop down menu -> Quick Run

