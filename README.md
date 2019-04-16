## Contributing

Any member of the organization is welcome to contribute to the blog, not only
collaborators. The main branch that is served over `blog.image-rs.org` should
only be changed through PRs with at least one review, regardless of the author.
Major changes to the structure or look should gather some feedback over a week
but at least three days (so that one is a work day).

Most pressing issues:

* Design the pages instead of relying on a rather unstructured default look.
* Establish a system for tags and categories
* Evaluate how to present posts by member that do not concern the project
  itself, such as library usage reports, abstract ideas and visions, and
  educational content on image related topics. Possibly through a
  weekly/monthy/quarterly aggregation post and summary paragraphs?

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
