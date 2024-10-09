## Tercen developer's guide

This is the developer's guide to Tercen.

The online version can be found at [https://tercen.github.io/developers_guide/](https://tercen.github.io/developers_guide/).



```shell
sudo add-apt-repository ppa:deadsnakes/ppa

export UID=$(id -u)
export GID=$(id -g)

docker run --rm -it \
   --user $UID:$GID \
   -v /etc/group:/etc/group:ro \
   -v /etc/passwd:/etc/passwd:ro \
   -v /etc/shadow:/etc/shadow:ro \
   -v ~/:/home/$USER \
   --workdir="/home/$USER/dev/tercen/developers_guide" \
   python:bullseye bash
   
docker run --rm -it \
   --user $UID:$GID \
   -v /etc/group:/etc/group:ro \
   -v /etc/passwd:/etc/passwd:ro \
   -v /etc/shadow:/etc/shadow:ro \
   -v ~/dev/tercen/developers_guide:/home/$USER/dev/tercen/developers_guide \
   --workdir="/home/$USER/dev/tercen/developers_guide" \
   python:bullseye bash
   
docker run --rm -it \
   -v ~/dev/tercen/developers_guide:/home/$USER/dev/tercen/developers_guide \
   --workdir="/home/$USER/dev/tercen/developers_guide" \
   python:bullseye bash

PATH=/home/alex/.local/bin:$PATH

pip install -r script/requirements_docs.txt

export GIT_COMMITTER_NAME=ci-bot
export GIT_COMMITTER_EMAIL=ci-bot@tercen.com

mike delete --all
mike deploy --push --branch gh-pages 1.0

mike deploy --remote origin --push --branch gh-pages

pip install mkdocs
pip install mike


mkdocs new documentation

cd documentation
mkdocs serve
mkdocs build

echo "site/" >> .gitignore


python3 -m http.server

find . -name "*.t1" -exec bash -c 'mv "$1" "${1%.t1}".t2' - '{}' +

find . -name "*.Rmd" -exec bash -c 'mv "$1" "${1%.Rmd}".md' - '{}' +

find . -name "*.Rmd" -exec bash -c 'mv "$1" "${1%.Rmd}".md' - '{}' \;


docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material

docker run --rm -it \
   -p 8000:8000 \
   -v ${PWD}:/docs \
   -e MKDOCS_GIT_COMMITTERS_APIKEY=$GITHUB_TOKEN \
   --entrypoint sh squidfunk/mkdocs-material

pip install mike
pip install mkdocs-git-committers-plugin-2
pip install mkdocs-git-revision-date-localized-plugin
#pip install mkdocs-git-authors-plugin
 
mkdocs serve --dev-addr=0.0.0.0:8000
 
```