---
title: "How I created this blog"
date: 2020-10-04 16:08:00 -0400
categories: [tutorial]
pin: false
---

In this post, I am going to show you how I created this blog. I wanted something that was easy to maintain and easy on the wallet.
Using the same approach, you can start you own blog entirely for free. There are some things that will cost money, but these are entirely optional. You can choose to use a premium Jekyll theme (more on these later), and you can register your own custom domain.

## Jekyll

This blog is built using [Jekyll](https://jekyllrb.com/). Jekyll is a framework used to generate static sites. It allows you to focus on the content, and not the aesthetics and organization of the site itself. I am not very adept at designing websites, so Jekyll is the perfect choice for me because Jekyll has a [theme system](https://jekyllrb.com/docs/themes/) that allows you to change the look and feel of your blog very easily.

Some themes will cost money, but there are plenty out there that are free. The theme I am using here is called [Chirpy](https://jekyll-themes.com/chirpy/). I picked this theme because I like the sidebar and fact that you can switch between a light and dark mode.

I followed the [instructions](https://github.com/cotes2020/jekyll-theme-chirpy#installation) on installing the theme, which just involves forking the repo and cloning it to your local.

I am going to defer to the [Jekyll documentation](https://jekyllrb.com/docs/posts/) on how to add a post.

In order to view you blog locally, you will need to install Jekyll which is available as a Ruby Gem. Instructions on how to do that can be found [here](https://jekyllrb.com/docs/installation/).

After you have done that you can run

```
bundle install
bundle exec jekyll serve
```

What `Jekyll` will do is take your files and build them into `_sites`, it then serves them at `localhost:4000`.

## GitHub Pages

At this point, you have a Jekyll built site and now we want to publish it for the world to see.
[GitHub Pages](https://pages.github.com/) is a feature of GitHub where GitHub will host sites for your projects.
Each GitHub account gets one site, and each of your repositories gets a site as well.
So this blog is available at `tanjacky.github.io`, and if I wanted to create a site for, say `test-repository`, GitHub also provides a site at `tanjacky.github.io/test-repository`.

For simple Jekyll sites, it is sufficient enough to just push to the repository. GitHub has built in support for Jekyll so it can build your site and serve the files under `_site`, similar to how you view your site locally.

If, like me, where you want to have more control of the build process, you can use GitHub Actions.

### GitHub Actions

[GitHub Actions](https://github.com/features/actions) are automated tasks that you can run on your repository. They charge you according to the amount of time you use, but GitHub provides 2000 minutes per month for free.

There is [documentation](https://jekyllrb.com/docs/continuous-integration/github-actions/) on Jekyll which outlines how to use GitHub Actions to build your site.

However, the Jekyll theme I am using requires some custom scripts to run, so I could not follow that guide completely.

You can see my [workflow](https://github.com/tanjacky/tanjacky.github.io/blob/master/.github/workflows/deploy.yml) here.

First, I checkout the repository, then set up Ruby.

```yml
- uses: actions/checkout@v2

- uses: actions/setup-ruby@v1
        with:
          ruby-version: 2.7.x
```

Next, we use the `cache` action to cache our build dependencies. This is useful because not only does it speed up our builds, but we reduce the amount of minutes used out of our free 2000 minute/month allotment.

```yml
- uses: actions/cache@v2
  id: bundle-cache
  with:
    path: vendor/bundle
    key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
    restore-keys: |
     ${{ runner.os }}-gems-

- name: Bundle config
  run: |
    bundle config path vendor/bundle

- name: Bundle Install
        if: steps.bundle-cache.outputs.cache-hit != 'true'
        run: |
          bundle install

- name: Bundle Install locally
        if: steps.bundle-cache.outputs.cache-hit == 'true'
        run: |
          bundle install --local
```

The Chirpy theme comes with a script to build the site. `-d` tells it to output the files to `/build`.

```yml
- name: Build Site
  run: |
  bash tools/build.sh -d build
```

The final step is to push the outputted files back to the repository.

```yml
- name: Push
  run: |
    cd build
    remote_branch=gh-pages
    remote_repo="https://${{ secrets.JEKYLL_PAT }}@github.com/$GITHUB_REPOSITORY.git"
    git init
    git config user.name "$GITHUB_ACTOR"
    git config user.email "$GITHUB_ACTOR@users.noreply.github.com"
    git add .
    git commit -m "jekyll build from Action $GITHUB_SHA"
    git push --force $remote_repo master:$remote_branch
```

In order for GitHub actions to push code back into your repository, you'll need to create a [Personal Access Token](https://github.blog/2013-05-16-personal-api-tokens/) with `public_repo` access (assuming your repo is public, it will need `repo` access if it is private). You then need to set that value as a GitHub Secret within your repo. You can then reference that secret within your workflow. I set the secret name as `JEKYLL_PAT`, so I can reference the value with `secrets.JEKYLL_PAT`

This will push the output files to the `gh-pages` branch. Then, in the `Settings` tab of my repository, I set `gh-pages` as the source of the GitHub page.
![source](/assets/img/posts/how-i-created-this-blog/source.png)

## Custom Domain

This step is optional, but having a custom domain does make your blog look more cool and professional (in my opinion).

There are a variety of companies you can go use to register your domain name, pick whichever one you want. I went with [Namecheap](https://www.namecheap.com) because they offered `jackytan.dev` to me at a lower price than the other competitors.

Once you registered your domain, you need to set a `ALIAS` record.
![alias](/assets/img/posts/how-i-created-this-blog/alias.png)

This will point your domain to your GitHub page.

Next you will need to update your respository with your custom domain.
![custom-domain](/assets/img/posts/how-i-created-this-blog/custom-domain.png)

This will create a `CNAME` record for your GitHub page to your custom domain, so when I go to [tanjacky.github.io](https://tanjacky.github.io), the URL will get mapped to `jackytan.dev`.

That's how I created this blog, hope you were able to learn something from this.

Thanks for reading, take care.

![o.o](/assets/img/milo/o.o.jpg)
