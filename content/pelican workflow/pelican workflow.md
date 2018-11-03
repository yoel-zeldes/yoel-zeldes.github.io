Title: Pelican and GitHub Pages workflow
Slug: pelican-and-github-pages-workflow
Date: 2016-11-08 21:00
Tags: pelican, git
Summary: A simple worflow for writing posts using Pelican and GitHub Pages.
header_cover: images/pelican-and-github-pages-workflow/cover.png

This blog is powered by [Pelican](http://docs.getpelican.com/en/stable) and hosted through GitHub using [GitHub Pages](https://pages.github.com/).
In this post I'll describe the workflow I use when deploying new posts.

For those of you not familiar with these technologies, Pelican is a static site generator - meaning you can write your content in a format such
as [Markdown](http://daringfireball.net/projects/markdown), and Pelican will automatically generate the HTML files for you; and GitHub Pages is
a service provided by GitHub for hosting a website under the <your-username\>.github.io URL.

Using Pelican and GitHub Pages is quite easy. There's one annoying little thing though... GitHub Pages assumes
the `master` branch contains the root folder to be served to the world. If you're using Pelican's default settings, the `output`
folder is the folder you want to serve. `output` contains the generated website's files.
A natural choice of how to organize the files inside the repository
would be to define pelican's root folder - the parent of `output` - as the root of the repository. But GitHub Pages needs
`output` to be the root. Bummer...

There are [those](http://mavant.com/blog/2014/03/10/pelican-git-hooks-github-dot-io)
who solve it using two separate repositories: one for the website "source" files, and
one for the output which will be served using GitHub Pages.

I personally don't like breaking my blog into two repositories. I want to keep everything in one place, so I chose to solve the problem
using branches and git hooks.

The first step is to create two branches:

* `source` will contain the blog's "source" files, namely - all the files such as the `content` folder which contains the actual posts, and `pelicanconf.py` file.
* `master` will contain only the contents of `output`.

These branches will obviously live in my GitHub Pages repo
([https://github.com/yoel-zeldes/yoel-zeldes.github.io](https://github.com/yoel-zeldes/yoel-zeldes.github.io)),
and since the `master` branch contains `output`'s contents, a user navigating to [yoel-zeldes.github.io](yoel-zeldes.github.io)
will be presented with the goodness of my blog.

Unfortunately, manually maintaining these two branches is cumbersome. Git hooks to the rescue!

Git has a mechanism to execute custom scripts when certain important actions occur. In my case, whenever I push a commit to the `source` branch,
I'd like the `master` branch to get updated with the new contents of `output`. This can be done using the pre-push hook, which is executed -
you guessed it - just before a push occurs.

All you have to do is create a file named `.git/hooks/pre-push` with the following content:

	:::bash
	#!/bin/sh
	while read local_ref local_sha remote_ref remote_sha
	do
			if [ "$remote_ref" = "refs/heads/source" ]
			then
					echo 'pushing output folder (production version) to master...'
					pelican content -o output -s publishconf.py
					echo anotherdatum.com > output/CNAME
					ghp-import output
					git push --no-verify git@github.com:yoel-zeldes/yoel-zeldes.github.io.git gh-pages:master
					pelican content -o output
			fi
	done

	exit 0

1. The first thing the script does is iterating over the commits that are about to be pushed. Specifically, only commits that are pushed to the
`source` branch are of interest to us.
2. If commits are pushed to `source`, it executes `pelican` command using `publishconf.py`. This will generate the production version of the blog into `output`.
3. It then creates a `CNAME` file, which is needed since I use a custom domain ([http://anotherdatum.com](http://anotherdatum.com)).
4. The [GitHub Pages Import](https://github.com/davisp/ghp-import) tool is used for copying the contents of `output` to a branch named `gh-pages`.
5. `gh-pages` is pushed to the remote `master` branch. `--no-verify` skips the pre-push hook so this script won't run again.
6. `pelican` is executed again to generate the development version of my blog, so I'll be able to write the next post.

Now, whenever I push to `source`, and only to `source`, `master` gets updated with the new contents. Cool!

One last small detail: I added `output` to the `.gitignore` file. This way, the `source` branch won't include this folder.
We don't really want to put it under version control - it would be like putting other types of generated files such as
`.pyc` or `.o` under version control.
