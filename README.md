## Tercen developer's guide

This is the developer's guide to Tercen.

The online version can be found at [https://tercen.github.io/developers_guide/](https://tercen.github.io/developers_guide/).

## Build

```shell
docker run --rm -it \
   -p 8000:8000 \
   -v ${PWD}:/docs \
   -e MKDOCS_GIT_COMMITTERS_APIKEY=$GITHUB_TOKEN \
   --entrypoint sh \
   squidfunk/mkdocs-material

pip install mike
pip install mkdocs-git-committers-plugin-2
pip install mkdocs-git-revision-date-localized-plugin
 
mkdocs serve --dev-addr=0.0.0.0:8000
```