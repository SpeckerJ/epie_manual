# Welcome to MkDocs


For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.fd
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.
* `mkdocs serve --livereload` - Activate live reload in case it is not working

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.


## Commit to github

1. Save your file in VS Code
2. Stage the change `git add .`

3. Commit the change, including a message
`git commit -m "MESSAGE"`

4. Push to GitHub (main branch)
`git push origin main`

5. Rebuild the site
`mkdocs build`

6. Deploy to GitHub Pages
`mkdocs gh-deploy --force`

7. Wait 1-2 minutes, then open your site. If it is not updating, check the site in incognito mode (no cache)