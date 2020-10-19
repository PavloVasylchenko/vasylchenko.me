---
title: "Getting started with Hugo"
date: 2020-09-27T19:47:36Z
categories: 
    - "Manuals"
draft: false
---

Building a blog with Hugo.
Recently, static sites have become very popular.
This is most likely because they are quite easy to maintain and there are many platforms which can be used for publishing them.
For example, if earlier it was necessary to work with HTML to create static sites, now knowledge of markdown will be enough for most of tasks.
Hugo sites are great for personal pages when you don't need to update content frequently.
Hugo is a set of tools that allows you to generate a static site from markdown files and templates.
I also really like the presence of docker images that can create and assemble a site without installing Go, Hugo and other components.

So, let's begin.
If you have Linux with Docker installed (https://www.docker.com), you need to follow only a few steps.
Create an empty site in the vasylchenko.me folder with this command.
`docker run --rm -it -v $(pwd)/:/src klakegg/hugo:ext new site vasylchenko.me`
Next, go to this folder with the `cd vasylchenko.me` command.
After that, we need to create a Git repository for version control.
For this, execute the command `git init`.

It's time to choose a theme. You can see a list of available popular themes by the link https://themes.gohugo.io
For instance I chose the theme https://themes.gohugo.io/devise/ . 
Follow the homepage link and then run the command
`git submodule add https://github.com/austingebauer/devise themes/devise` to install the theme.
This command will add a theme from the repository to the themes folder of our site.

Thus, we have a folder with the vasylchenko.me website and a folder with the vasylchenko.me/themes/devise theme.
Now you need to add a line to the configuration file indicating the selected theme.
This can be done by editing the file or by running the command
`echo 'theme = "devise"' >> config.toml`

Finally, lets assemble the site and check the result.
`docker run --rm -it -v $(pwd):/src klakegg/hugo:ext`
Now you can go to the vasylchenko.me/public folder and open the index.html file in your web browser.
If the site looks as expected it can be uploaded to static hosting.
