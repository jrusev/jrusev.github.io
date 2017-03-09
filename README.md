jrusev.github.io
================

This is the source code for my blog at [jrusev.github.io](http://jrusev.github.io/).
The blog is built with [Hugo](http://gohugo.io/) and the generated HTML is in
the `master` branch.

## How to build the site

GitHub allows [one site](https://pages.github.com/) per user and serves it
automatically from a repository named `username.github.io`, where *username* is
your username on GitHub. So we will keep the HTML of the site in the `master`
branch of the repo and the source code in the `source` branch (this branch).

This is how to create a new post and make it live using Hugo:

  1. Create a new Markdown file in the `content/post` directory. You can either
     copy an old post or let Hugo create it with default metadata (let's use
     [casper](http://themes.gohugo.io/casper/) as a theme):

     ```bash
     $ hugo new -t casper post/my-post.md
     ```

  * Edit your post content in
    [Markdown](https://guides.github.com/features/mastering-markdown/) with your
    [favorite](https://atom.io/) text editor. You can put images to
    `static/images` folder and reference them in you post like so:

    ```markdown
    Check the photo I took yesterday:

    ![Image of Sunrise](/images/sunrise.jpg)
    ```

  * See how your post looks like by running a Hugo server locally:

    ```bash
    # --renderToDisk is needed to prevent hugo server from hanging on 32-bit
    # Windows until Hugo v0.16 is released
    $ hugo server --renderToDisk
      3 pages created
      in 51 ms
      Watching for changes in {data,content,layouts,static,themes}
      Serving pages from \public
      Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
      Press Ctrl+C to stop
    ```

  * When you are done, you can commit your changes to the `source` branch on
    GitHub.

  * Now you are ready to publish your new post. First, you need to generate the
    HTML for your site with Hugo:

      ```bash
      $ hugo
      ```

  * Your site HTML is ready inside the `public` folder. Go inside that folder
    and push to your `master` branch on GitHub:

    ```bash
    $ cd public
    $ git add .
    $ git commit -m "Add new post"
    $ git push origin master
    ```

  * Your new content should be live shortly - fire up a browser and go to
  [http://jrusev.github.io](http://jrusev.github.io).
  
## Automatic build and deployment

You can automate the publish step (and avoid pushing to two branches) if you host your site on [GitLab](https://pages.gitlab.io/). You will need to add a `.gitlab-ci.yml` file to the root directory of your repository ([example](https://gitlab.com/pages/hugo/blob/master/.gitlab-ci.yml)), and configure your GitLab project to use a [Runner](https://docs.gitlab.com/ce/ci/runners/README.html). Then each commit or push, will trigger your CI pipeline (`pages` is a special job that is used to upload static content to GitLab that can be used to serve your website). With GitLab you can still have two branches and setup CI in your master branch (where your static site lives) - see the [Jekyll](https://gitlab.com/pages/jekyll-branched/tree/master) project.
