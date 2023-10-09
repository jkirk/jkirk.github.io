---
title: "How to deploy a Hugo Site as GitHub Page"
created: 2023-08-02T00:12:17+0200
date: 2023-08-08T17:31:59+0200
modified: 2023-09-15T01:33:50+0200
---

A couple of years ago, I found a tutorial that showed how to use [Jekyll](https://jekyllrb.com/) to set up GitHub pages.
I never got very far. This guide shows how to use [Hugo](https://gohugo.io/) to do the same thing.
<!--more-->

## Introduction

A few years ago I found the blog "[Using Jekyll, Asciidoctor and GitHub Pages for Static Site Creation](https://yermilov.github.io/blog/2017/02/20/using-jekyll-asciidoctor-and-github-pages-for-static-site-creation/)" from [@yermilov](https://github.com/yermilov).
It inspired me to start and set up my own Jekyll blog on GitHub Pages.
I followed @yermilov's quick start guide, created a simple Jekyll site and deployed it on my GitHub Page.

This should and could have been the start of my blogging career! (it was not... 😉)

But well, this was five years ago and the "Your awesome title" site stayed there for a very long time:

![GitHub Pages Deployments / History](screenshot_20230802T031131.png "My GitHub Pages Deployments / History")

Anyway, the blog was made with [Jekyll](https://jekyllrb.com/) and in the meantime [Hugo](https://gohugo.io/) caught my interest.

First, I had to figure out how to deploy the Hugo site to my GitHub Page.

I read some documentation before I started my journey:

* [GitHub Pages](https://pages.github.com/)
* [Hugo | Host on GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

My Jekyll GitHub Page was in the branch `gh-pages`.
I wanted to put the Hugo site in a separate `gh-pages-hugo` branch, and so I as a first step I created a new unrelated branch:

```sh
  ~/projects/websites/jkirk.github.io on  gh-pages (2cb49e6) via 💎 v2.7.4 took 35s
  at 2023-08-02 01:24:04 +02:00 ❯ git checkout --orphan gh-pages-hugo
  Switched to a new branch 'gh-pages-hugo'

  at 2023-08-02 01:24:11 +02:00 ❯ git rm -rf .
  rm 'Gemfile'
  rm '_config.yml'
  rm '_posts/2017-02-10-welcome-to-mini-jekyll.md'
  rm 'about.md'
  rm 'index.md'
```

## Set up Hugo and the initial Hugo site

I am still running Debian/bullseye, which only includes Hugo in version [0.80.0](https://packages.debian.org/bullseye/hugo).
Debian/bookwork includes Hugo in version [0.111.3](https://packages.debian.org/bookworm/hugo), but my Debian upgrade is still on my to-do list.
So I decided to use the docker/podman image `hugo:0.111.3-debian` to match the Hugo version in Debian/bookworm to develop the site:

```sh
  at 2023-08-02 01:32:51 +02:00 ❯ podman pull docker.io/klakegg/hugo:0.111.3-debian
  Trying to pull docker.io/klakegg/hugo:0.111.3-debian...
  Getting image source signatures
  Copying blob 4f4fb700ef54 done
  Copying blob f03b40093957 done
  Copying blob 09681a3abe6b done
  Copying blob b06ae5579247 done
  Copying blob 8b22c822b712 done
  Copying config e346f88099 done
  Writing manifest to image destination
  Storing signatures
  e346f880998840e51161af8ea35dc593b369bc2b32373c854deffe327b5c593e
  podman pull docker.io/klakegg/hugo:0.111.3-debian  12.15s user 4.41s system 64% cpu 25.814 total
```

I then created a new site by following the quick start guide:
https://gohugo.io/getting-started/quick-start/

```sh
  ❯ podman run --rm -it -v $(pwd):/src docker.io/klakegg/hugo:0.111.3-debian new site . --force
  Congratulations! Your new Hugo site is created in /src.

  Just a few more steps and you're ready to go:

  1. Download a theme into the same-named folder.
     Choose a theme from https://themes.gohugo.io/ or
     create your own with the "hugo new theme <THEMENAME>" command.
  2. Perhaps you want to add some content. You can add single files
     with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
  3. Start the built-in live server via "hugo server".

  Visit https://gohugo.io/ for quickstart guide and full documentation.
```

I browsed the [Hugo Themes](https://themes.gohugo.io/), selected the tag `blog`, and at first chose the theme [Nightfall](https://themes.gohugo.io/themes/hugo-theme-nightfall/):

```sh
  ❯ git submodule add https://github.com/LordMathis/hugo-theme-nightfall.git themes/nightfall
  Cloning into '/tmp/tmp.botstjfZla/jkirk.github.io/themes/nightfall'...
  remote: Enumerating objects: 350, done.
  remote: Counting objects: 100% (341/341), done.
  remote: Compressing objects: 100% (178/178), done.
  remote: Total 350 (delta 160), reused 297 (delta 143), pack-reused 9
  Receiving objects: 100% (350/350), 996.14 KiB | 1.06 MiB/s, done.
  Resolving deltas: 100% (160/160), done.

  ❯ podman run --rm -it -v $(pwd):/src docker.io/klakegg/hugo:0.111.3-debian server
  Start building sites …
  hugo v0.111.3-5d4eb5154e1fed125ca8e9b5a0315c4180dab192 linux/amd64 BuildDate=2023-03-12T11:40:50Z VendorInfo=hugoguru
  WARN 2023/08/03 17:58:45 found no layout file for "HTML" for kind "taxonomy": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
  WARN 2023/08/03 17:58:45 found no layout file for "HTML" for kind "home": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
  WARN 2023/08/03 17:58:45 found no layout file for "HTML" for kind "taxonomy": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.

                     | EN
  -------------------+-----
    Pages            |  3
    Paginator pages  |  0
    Non-page files   |  0
    Static files     |  0
    Processed images |  0
    Aliases          |  0
    Sitemaps         |  1
    Cleaned          |  0

  Built in 21 ms
  Watching for changes in /src/{archetypes,assets,content,data,layouts,static}
  Watching for config changes in /src/config.toml
  Environment: "DEV"
  Serving pages from memory
  Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
  Web Server is available at http://localhost:1313/ (bind address 0.0.0.0)
  Press Ctrl+C to stop
```

Port 1313 is [exposed](https://github.com/klakegg/docker-hugo/blob/0.111.3/src/docker/debian/base.df) in the Docker image, but I could not access [http://127.0.0.1:1313]().
With Podman I had to specify the port to expose it from the container with the `-p` option[^1]:

[^1]: https://stackoverflow.com/a/69885042/2142030

```sh
  at 2023-08-02 02:08:27 +02:00 ❯ podman run --rm -it -v $(pwd):/src -p 1313:1313 docker.io/klakegg/hugo:0.111.3-debian server
  Start building sites …
  [...]
```

Then I got "Page Not Found". It turned out that I had forgotten to add `theme = 'nightfall'` to `config.toml` and that the theme was not compatible with the this Hugo version:

```sh
  ❯ echo "theme = 'nightfall'" >> config.toml

  ❯ podman run --rm -it -v $(pwd):/src -p 1313:1313 docker.io/klakegg/hugo:0.111.3-debian server
  WARN 2023/08/03 18:02:12 Module "nightfall" is not compatible with this Hugo version; run "hugo mod graph" for more information.
  Start building sites …
  hugo v0.111.3-5d4eb5154e1fed125ca8e9b5a0315c4180dab192 linux/amd64 BuildDate=2023-03-12T11:40:50Z VendorInfo=hugoguru
  Error: Error building site: TOCSS: failed to transform "sass/main.scss" (text/x-scss). Check your Hugo installation; you need the extended version to build SCSS/SASS with transpiler set to 'libsass'.: this feature is not available in your current Hugo version, see https://goo.gl/YMrWcn for more information
  Built in 18 ms
```

I briely investigated why the Hugo version is incomptible with this theme:
https://gohugo.io/troubleshooting/faq/#i-get-this-feature-is-not-available-in-your-current-hugo-version

The extended version is required for the theme, but the docker image does not have the extended version installed:

```sh
  at 2023-08-02 02:44:05 +02:00 ❯ podman run --rm -it -v $(pwd):/src -p 1313:1313 docker.io/klakegg/hugo:0.111.3-debian version
  hugo v0.111.3-5d4eb5154e1fed125ca8e9b5a0315c4180dab192 linux/amd64 BuildDate=2023-03-12T11:40:50Z VendorInfo=hugoguru

  at 2023-08-02 02:46:38 +02:00 ❯ cat themes/nightfall/config.toml
  [module]
    [module.hugoVersion]
      extended = true
      min = "0.80.0"
```

I then chose the theme [Binario](https://themes.gohugo.io/themes/binario/).
I removed the `themes/nightfall` submodule and added the `themes/binario` submodule and changed the theme to `theme = 'binario'` in `config.toml`:

```sh
  ❯ git submodule deinit --force themes/nightfall
  Cleared directory 'themes/nightfall'
  Submodule 'themes/nightfall' (https://github.com/LordMathis/hugo-theme-nightfall.git) unregistered for path 'themes/nightfall'

  ❯ git submodule add https://github.com/Vimux/Binario.git themes/binario
  Cloning into '/tmp/tmp.botstjfZla/jkirk.github.io/themes/binario'...
  remote: Enumerating objects: 1640, done.
  remote: Counting objects: 100% (87/87), done.
  remote: Compressing objects: 100% (45/45), done.
  remote: Total 1640 (delta 43), reused 70 (delta 36), pack-reused 1553
  Receiving objects: 100% (1640/1640), 1.38 MiB | 1.09 MiB/s, done.

  ❯ sed -i -e "s/nightfall/binario/" config.toml
  ❯ grep theme config.toml
  theme = 'binario'

  ❯ podman run --rm -it -v $(pwd):/src -p 1313:1313 docker.io/klakegg/hugo:0.111.3-debian server
  Start building sites …
  hugo v0.111.3-5d4eb5154e1fed125ca8e9b5a0315c4180dab192 linux/amd64 BuildDate=2023-03-12T11:40:50Z VendorInfo=hugoguru

                     | EN
  -------------------+-----
    Pages            |  7
    Paginator pages  |  0
    Non-page files   |  0
    Static files     | 13
    Processed images |  0
    Aliases          |  3
    Sitemaps         |  1
    Cleaned          |  0

  Built in 41 ms
  Watching for changes in /src/{archetypes,assets,content,data,layouts,static,themes}
  Watching for config changes in /src/config.toml
  Environment: "DEV"
  Serving pages from memory
  Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
  Web Server is available at http://localhost:1313/ (bind address 0.0.0.0)
  Press Ctrl+C to stop
```

Finally I was ready to create my post (I write here):

```sh
  ❯ podman run --rm -it -v $(pwd):/src docker.io/klakegg/hugo:0.111.3-debian new posts/my-first-post.md
  Content "/src/content/posts/my-first-post.md" created
```

The file looked something like this:

    ---
    title: "How to deploy a Hugo Site as GitHub Page"
    date: 2023-08-02T00:12:17Z
    draft: true
    ---

To show the draft while working I ran the Hugo server with the options `-D --navigateToChanged`:

```sh
  ❯ podman run --rm -it -v $(pwd):/src -p 1313:1313 docker.io/klakegg/hugo:0.111.3-debian server -D --navigateToChanged
  Start building sites …
  hugo v0.111.3-5d4eb5154e1fed125ca8e9b5a0315c4180dab192 linux/amd64 BuildDate=2023-03-12T11:40:50Z VendorInfo=hugoguru

                     | EN
  -------------------+-----
    Pages            | 10
    Paginator pages  |  0
    Non-page files   |  1
    Static files     | 13
    Processed images |  0
    Aliases          |  4
    Sitemaps         |  1
    Cleaned          |  0

  Built in 144 ms
  Watching for changes in /src/{archetypes,assets,content,data,layouts,static,themes}
  Watching for config changes in /src/config.toml
  Environment: "DEV"
  Serving pages from memory
  Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
  Web Server is available at http://localhost:1313/ (bind address 0.0.0.0)
  Press Ctrl+C to stop
```

## Publish the Hugo site to my GitHub Pages

I now was ready to publish the Hugo site to my GitHub pages.

I pushed the branch to GitHub:

```sh
  ❯ git push origin gh-pages-hugo
  Enumerating objects: 10, done.
  Counting objects: 100% (10/10), done.
  Delta compression using up to 8 threads
  Compressing objects: 100% (8/8), done.
  Writing objects: 100% (10/10), 1.09 KiB | 560.00 KiB/s, done.
  Total 10 (delta 1), reused 0 (delta 0), pack-reused 0
  remote: Resolving deltas: 100% (1/1), done.
  remote:
  remote: Create a pull request for 'gh-pages-hugo' on GitHub by visiting:
  remote:      https://github.com/jkirk/jkirk.github.io/pull/new/gh-pages-hugo
  remote:
  To github.com:jkirk/jkirk.github.io.git
   * [new branch]      gh-pages-hugo -> gh-pages-hugo
```

In Settings > Pages > Build and deployment > Source I changed "Deploy from a branch" to "GitHub Actions".

I created `.github/workflows/hugo.yml` and set the branch to `gh-pages-hugo` and Hugo version to `HUGO_VERSION: 0.111.3` like described in Step 6 of [Host on GitHub Pages | Hugo](https://gohugo.io/hosting-and-deployment/hosting-on-github/).

I then committed and pushed the change:

```sh
  ❯ git log -1
  commit 7fbee53 (HEAD -> gh-pages-hugo)
  Author: Darshaka Pathirana <dpat@syn-net.org>
  Date:   2023-08-06 16:41:16 +0200

      Add GitHub Workflow to deploy Hugo site

      See: https://gohugo.io/hosting-and-deployment/hosting-on-github/

  ~/projects/websites/jkirk.github.io on  gh-pages-hugo (7fbee53) [!?]
  ✦ at 2023-08-06 16:41:57 +02:00 ❯ git push
  fatal: The current branch gh-pages-hugo has no upstream branch.
  To push the current branch and set the remote as upstream, use

      git push --set-upstream origin gh-pages-hugo


  ~/projects/websites/jkirk.github.io on  gh-pages-hugo (7fbee53) [!?]
  ✦ at 2023-08-06 16:42:02 +02:00 ❯ git push --set-upstream origin gh-pages-hugo
  Enumerating objects: 6, done.
  Counting objects: 100% (6/6), done.
  Delta compression using up to 8 threads
  Compressing objects: 100% (3/3), done.
  Writing objects: 100% (5/5), 1.39 KiB | 1.39 MiB/s, done.
  Total 5 (delta 1), reused 0 (delta 0), pack-reused 0
  remote: Resolving deltas: 100% (1/1), completed with 1 local object.
  To github.com:jkirk/jkirk.github.io.git
     0510547..7fbee53  gh-pages-hugo -> gh-pages-hugo
  Branch 'gh-pages-hugo' set up to track remote branch 'gh-pages-hugo' from 'origin'.
```

But I received an error:

![Github Workflow deploy error](screenshot_20230806T164310.png)

    Branch "gh-pages-hugo" is not allowed to deploy to github-pages due to environment protection rules.

It turned out that I had (long ago) limited the allowed branches to `gh-pages` + `master` only:

![Environment / Configure github-pages / Deployment branches](screenshot_20230806T165953.png)

I removed the given branches and added the `gh-pages*` pattern.

Then I selected the last failed workflow run and re-run the failed job.

## Configure the Hugo site

The final step was to configure the Hugo site and pubish the post.

I took the `Config.toml` example in the [Binaro Configuration](https://github.com/vimux/binario#configuration) documentation and adjusted it to [my needs](https://github.com/jkirk/jkirk.github.io/blob/gh-pages-hugo/config.toml).

I was not sure what "Params.Breadcrumb" was for:

![Breadcrumb](screenshot_20230808T165217.png)

`homeText` in `[Params.Breadcrumb]` defaults to `.Site.Title`. I changed it to "Main":

```toml
[Params.Breadcrumb]
    enable = true # Enable breadcrumb block globally
    homeText = "Main" # Home node text
```

The [Footer Social Icons](https://github.com/vimux/binario#footer-social-icons) are only shown in the footer of the posts.

I tried to find out what `schema` in `[Params]` means, but could not find it in the Binario documentation.
Changing the setting also had no visible effect. So I did not add it.

## Problems, questions and challenges

### Web Server is available at //localhost:1313/ (bind address 0.0.0.0)

`Web Server is available at //localhost:1313/ (bind address 0.0.0.0)` was shown:

```sh
  ❯ podman run --rm -it -v $(pwd):/src -p 1313:1313 docker.io/klakegg/hugo:0.111.3-debian server -D --navigateToChanged
  Start building sites …
  hugo v0.111.3-5d4eb5154e1fed125ca8e9b5a0315c4180dab192 linux/amd64 BuildDate=2023-03-12T11:40:50Z VendorInfo=hugoguru

                     | EN
  -------------------+-----
    Pages            | 10
    Paginator pages  |  0
    Non-page files   |  1
    Static files     | 13
    Processed images |  0
    Aliases          |  4
    Sitemaps         |  1
    Cleaned          |  0

  Built in 118 ms
  Watching for changes in /src/{archetypes,assets,content,data,layouts,static,themes}
  Watching for config changes in /src/config.toml
  Environment: "DEV"
  Serving pages from memory
  Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
  Web Server is available at //localhost:1313/ (bind address 0.0.0.0)
  Press Ctrl+C to stop
```

I had `baseURL = '/'` in `config.toml` for my initial tests:

    baseURL = '/'
    languageCode = 'en-us'
    title = 'Share Your Knowledge'
    theme = 'binario'

    [Author] # Used in authorbox
      name = "Darshaka Pathirana"

I changed it to `baseURL = 'https://jkirk.github.io/'`, which fixed it for me.[^2]

[^2]: https://github.com/gohugoio/hugo/issues/10765

### How do I insert images in my post (relative to the content)?

It was not obvious for me, where to save the images I want to insert in my post.

There are multiple options:

* Access the image as a page resource
* Access the image as a global resource

See: [Image processing | Hugo](https://gohugo.io/content-management/image-processing/#image-resources)

I wanted the put the images relative to the content and found a comment in the issue [gohugoio/hugo#1240](https://github.com/gohugoio/hugo/issues/1240#issuecomment-284418725) where images are saved as `my-post/image1.png` and referenced like this:

```
![image](image1.png)
```

This seems to be called [Leaf Bundle](https://gohugo.io/content-management/page-bundles/#leaf-bundles) in the Hugo documentation but without a `my-post/index.md` file.
Here the the structure looks like this and I am not sure why this works:

```
content/
├── posts
│   ├── my-post.md
│   ├── my-post/image1.png

```

Some more links:

* [How to insert an image in my post on Hugo? - Stack Overflow](https://stackoverflow.com/a/71506386/2142030)
* [Page resources | Hugo](https://gohugo.io/content-management/page-resources/)

### How to set a featured image for a page

I did not use it for this post, but I stumbled across this links, which may come in handy some time: [Featured Images](https://binario.netlify.app/post/featured-images/)

### How to add a subtitle / description below the title

While releasing the page, I had to write a descrtiption for my blog.
I choose:

> "Problems need to be solved. Solutions should be shared."

I wanted this line to be added below the title, but in the end decided to put it in the footer above the copyright line.

I had to override the themes's partial template `footer.html` and added the `Params.Subtitle` into it.

```sh
  ❯ cp themes/binario/layouts/partials/footer.html layouts/partials/footer.html


  ❯ git diff --no-index themes/binario/layouts/partials/footer.html layouts/partials/footer.html
  diff --git themes/binario/layouts/partials/footer.html layouts/partials/footer.html
  index 85a9f26..ee6e4f1 100644
  --- themes/binario/layouts/partials/footer.html
  +++ layouts/partials/footer.html
  @@ -1,5 +1,6 @@
   <footer class="footer">
          {{- partial "footer_social.html" . }}
          {{- partial "footer_menu.html" . }}
  +       <div class="footer__copyright">{{ .Site.Params.Subtitle }}<span class="footer__copyright-credits"></span></div>
          <div class="footer__copyright">© {{ now.Format "2006" }} {{ .Site.Params.copyright | default .Site.Title }}. <span class="footer__copyright-credits">{{ T "footer_credits" | safeHTML }}</span></div>
  -</footer>
  \ No newline at end of file
  +</footer>
```

See: https://gohugo.io/templates/partials/

### How to add the last modified date

While modifying this post, I wondered how to add the date of the last modification.
It was quite simple, I just had to add `modified:` to the page header.

See:

* https://gohugo.io/variables/page/
* https://gohugo.io/getting-started/configuration/#configure-dates

### How to set the content summaries

I noticed that the content summaries where too long.

There three summary splitting options:

- By default, Hugo automatically takes the first 70 words of your content as its summary and stores it into the .Summary page variable for use in your templates.
- Alternatively, you may add the `<!--more-->` summary divider where you want to split the article.
- You might want your summary to be something other than the text that starts the article. In this case you can provide a separate summary in the summary variable of the article front matter.

See: https://gohugo.io/content-management/summaries/
