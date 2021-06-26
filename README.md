# nathan-tw.github.io
My [personal website](https://nathan-tw.github.io) bootstrapped by [hugo](https://gohugo.io/), a static website generating engine. Also, github actions for CI/CD is triggered when a new article is published
, that is, new commit is pushed.

:octocat: github actions flow:

1. edit article on master branch.
2. trigger CI/CD pipeline, replace the gh-pages branch with the new build site.
3. deploy to github pages.
