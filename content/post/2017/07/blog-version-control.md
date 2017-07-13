---
title: "Blog Version Control"
date: 2017-07-13T21:39:38+10:00
---

Following on from [my last post](/2017/07/package-blog/)...

The Hugo [Quickstart Guide](https://gohugo.io/overview/quickstart/) gives instructions on setting up version control and publishing a site on GitHub Pages - but using the method of pushing to a *gh-pages* branch.

This is the typical path for a "project site", but a [user site](https://pages.github.com/#user-site) (with no repository in the URL) will publish to Pages only from the *master* branch. So adding the hugo files to version control required me to either start another project - or just be sensible about it and use a non-master branch of the user project (which must be named \<username\>.github.io).

The process was quick and simple (starting from public/, where I had just pushed the blog files to *master*):
```
$ cd ..
$ echo /public/ >> .gitignore
$ echo /themes/ >> .gitignore
$ git init
$ git remote add origin https://github.com/forfuncsake/forfuncsake.github.io.git
$ git checkout -b hugo
$ git add --all .
$ git commit
$ git push -u origin hugo
```

For any future updates, like this one, I now simply need to:
```
$ hugo
$ git add --all .
$ git commit
$ git push origin hugo
$ cd public
$ git add --all .
$ git commit
$ git push origin master
```

Job done.

