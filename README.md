# MkDocs-blog

Just some place to publish my notes.

I'm following instructions at <https://www.mkdocs.org/user-guide/deploying-your-docs/#github-pages>.

TL;DR - `mkdocs gh-deploy` to publish. Or add something like

```text
hbarta@rocinante:~/Documents/MkDocs-blog$ cat .git/hooks/pre-push
mkdocs gh-deploy
exit 0
hbarta@rocinante:~/Documents/MkDocs-blog$ 
```

Page is available at <https://HankB.github.io/MkDocs-blog/>
