## Serving locally

For basic usage, follow [the official Github guide][GHGuide]. If you are not
comfortable giving `sudo` to the `bundler` commands just for this project (i.e.
this is not your main project or you created a project related account to
confine access, ...) like me then you can also do:

1. Create some local directory used to hold the installed gems. I use
   `GEM_HOME=~/.local/lib/gems` to mirror the root tree that would otherwise be used.
2. > `bundle install --path $GEM_HOME`
3. Invoke jekyll through `bundle` as well
  > `bundle exec jekyll serve`


[GHGuide]: https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll
