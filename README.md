# DevOps Live #5

## Training

- Gitlab CI - https://ondrej-sika.cz/skoleni/gitlab-ci
- Terraform - https://ondrej-sika.cz/skoleni/terraform

## Repositories

- Setup of Gitlab using Terraform - https://github.com/ondrejsika/terraform-demo-gitlab
- Setup of Gitlab Runner in Docker - https://github.com/ondrejsika/ondrejsika-gitlab-runner
- Setup of Kubernetes Cluster using Terraform - https://github.com/ondrejsika/terraform-do-kubernetes-example

## Agenda

- Multiple Kubernetes Clusters
- Child Pipelines
- Generated Pipelines
- Multi Project Pipelines
- DAG & Needs

## Child Pipelines

[Docs](https://docs.gitlab.com/ce/ci/parent_child_pipelines.html)

```yaml
# .gitlab-ci.yml
pipeline_a:
  trigger:
    include: a/.gitlab-ci.yml
    strategy: depend
  only:
    changes:
      - a/**

pipeline_b:
  trigger:
    include: b/.gitlab-ci.yml
    strategy: depend
  only:
    changes:
      - b/**
```

```yaml
# a/.gitlab-ci.yml

stages:
  - build
  - deploy

build:
  stage: build
  script: echo Build service A

deploy:
  stage: deploy
  script: echo Deploy service A
```

```yaml
# b/.gitlab-ci.yml

stages:
  - build
  - deploy

build:
  stage: build
  script: echo Build service B

deploy:
  stage: deploy
  script: echo Deploy service B
```

## Generated Pipelines

```yaml
# .gitlab-ci.yml
stages:
  - generate
  - pipeline

generate:
  stage: generate
  image: python:3.7-slim
  script: python generate-gitlab-ci.py
  artifacts:
    paths:
      - .gitlab-ci.generated.yml

pipeline:
  stage: pipeline
  trigger:
    include:
      - artifact: .gitlab-ci.generated.yml
        job: generate
    strategy: depend
```

```python
# generate-gitlab-ci.py
import json

SERVICES = (
    "foo",
    "bar",
)

def make_service(name):
    return {
        "build %s" % name: {
            "stage": "build",
            "script":[
                "echo Build %s" % name,
            ]
        },
        "deploy %s" % name: {
            "stage": "deploy",
            "script":[
                "echo Deploy %s" % name,
            ]
        }
    }

with open(".gitlab-ci.generated.yml", "w") as f:
    pipeline = {}
    pipeline.update({
        "stages": ["build", "deploy"]
    })
    for service in SERVICES:
        pipeline.update(make_service(service))
    f.write(json.dumps(pipeline))
```
