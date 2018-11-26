---
title: Jekyll to hugo
date: 2018-06-28T23:30:00+09:00
tags: ["Blog"]
---

After reading [a really similar blog post](https://www.dannyguo.com/blog/migrating-from-jekyll-to-hugo/) 
about how Hugo is most simple and light compare to Jekyll,
I've decided to spend some time converting my old Github to a new modern Hugo blog.

Considering I'm just wishing a minimal and functional website,
after 30mn I had enough to consider a full migration.
Steps:

1. Install hugo with: **brew install hugo**
2. Import my old Jekyll website with: **hugo import ./pgreze.github.io ./pgreze.com**
3. Choose a theme: I've chosen [even](https://github.com/olOwOlo/hugo-theme-even) but you can choose any theme you like in the [offical themes listing](https://themes.gohugo.io/).
4. Use [Netlify](https://netlify.com) for hosting, reacting like travis-ci on every commits and rebuilding the website.
5. Change the site name and postpone to buy a real domain name.
6. Enjoy: [http://pgreze.netlify.com/](http://pgreze.netlify.com/)

Ok... Actually I faced some issues with Netlify, errors that you can discover in the 
[Deploys part](https://app.netlify.com/sites/pgreze/deploys/5b34ec94fdd72a50014bf497) 
of your account:

> 11:11:42 PM: Started building sites ...
> 11:11:42 PM: ERROR: 2018/06/28 14:11:42 template.go:477: template: /opt/build/repo/themes/even/layouts/_default/baseof.html:2: function "errorf" not defined
> 11:11:42 PM: ERROR: 2018/06/28 14:11:42 template.go:477: template: theme/partials/footer.html:41: function "now" not defined
> 11:11:42 PM: ERROR: 2018/06/28 14:11:42 template.go:477: template: theme/partials/post/outdated-info-warning.html:2: function "now" not defined
> 11:11:42 PM: Error: Error building site: yaml: unmarshal errors:
> 11:11:42 PM:   line 2: cannot unmarshal !!map into []map[string]interface {}

Seems like an old version, and that's exactly what you can avoid
by manually specifying a fixed version with
[a custom netlify.tml](https://www.netlify.com/blog/2017/04/11/netlify-plus-hugo-0.20-and-beyond/).

Now, I can:

1. Forget about Ruby implementation details like **bundle**
2. Not hack a multiple-branch based solution, because Github is not allowing to use plugins with Jekyll
3. Just edit directly with github.com and see the result after some time.
