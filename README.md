# MkDocs-blog

Just some place to publish my notes.

I'm following instructions at <https://www.mkdocs.org/user-guide/deploying-your-docs/#github-pages>.

TL;DR - `mkdocs gh-deploy` to publish. ~Or add something like~

```text
hbarta@rocinante:~/Documents/MkDocs-blog$ cat .git/hooks/pre-push
mkdocs gh-deploy
exit 0
hbarta@rocinante:~/Documents/MkDocs-blog$ 
```

`mkdocs gh-deploy` blows up if the repo can't be pushed for any reason.

NB: *Surprise!* `mkdocs gh-deploy` will publish any files in the local `.../docs` directory (at a minimum) even if they are not tracked, committed and pushed.

Page is available at <https://HankB.github.io/MkDocs-blog/>
