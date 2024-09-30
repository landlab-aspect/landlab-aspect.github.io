Source repository for the project website
====

Available online at https://landlab-aspect.github.io/

# Installation instructions for mkdocs

    pip install mkdocs-material

Alternatively you can use conda to install all packages into a new environment named `landlab-aspect-website` like this:

    conda env create -f environment.yml

Then the server can be run locally as

    mkdocs serve

and you should see something like

    INFO    -  Documentation built in 0.15 seconds
    INFO    -  [08:40:55] Watching paths for changes: 'docs', 'mkdocs.yml'
    INFO    -  [08:40:55] Serving on <http://127.0.0.1:8000/>
    INFO    -  [08:40:57] Browser connected: <http://127.0.0.1:8000/>
