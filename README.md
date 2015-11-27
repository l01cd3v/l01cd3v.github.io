# Published at [http://blog.l01cd3v.fr](http://blog.l01cd3v.fr)

    $ # Commit a new blog post
    $ # Run jekyll to create new _site files
    $ jekyll serve
    $ # Commit new _site on source branch
    $ git add _site
    $ git push origin source
    $ git subtree push --prefix _site origin master
