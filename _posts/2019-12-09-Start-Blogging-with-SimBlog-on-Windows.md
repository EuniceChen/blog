---
title: "Start Blogging with SimBlog on Windows"
---

I have a personal Windows PC. Usually I just play games on it, which went pretty well since a lot of games run only on Windows machine. When I'm at work, I code with Mac. With all the terminals and unix-like environment, coding on Mac was a great experience. Now I have a an idea to start blogging as a little after work entertainment, but this time I'll have to do it with my personal Windows PC.

When I started with this idea, I googled some suggestion on how to run a blog and this post caught my eye [Why Create Programming Blog](https://www.afternerd.com/blog/start-programming-blog/#why-create-programming-blog).

<div class="embed"><iframe src="https://www.afternerd.com/blog/start-programming-blog/#why-create-programming-blog" frameborder="0" allowfullscreen></iframe></div>

I liked some points within this post, like the reasons for writing blogs, but I didn't want to run my blog on a host I run. I want something fast and easy, that would let me start whenever I have something want to put down before my passion washed away. I looked at some different solutions like [Ghost](https://ghost.org/), [Medium](https://medium.com/) and [LogDown](http://logdown.com/). I saw different comments on these platforms and it seemed to be hard to find one that is free and codestyle-friendly. But I saw SimBlog at the end and decided to start with it. I am still learning and just started the experience with it. Maybe I can update more about the pros and cons of it later if my passion for blogging is not gone away in the future.

Anyway, this post is the my first one and it aims to both test SimBlog features and put down some notes for SimBlog configuration. I know SimBlog has a very thoroght setup manual [setup.md](https://github.com/puneetsl/simblog/blob/master/setup.md) on how to make everything work, but this post gives some extra information to help with the process to kick-off the SimBlog coding, it contains information like how to install Python and Atom along the way to configure SimBlog, assuming you don't have any coding tools installed on your Windows PC.

## Install Atom

The first thing I did was installing a good editor for SimBlog since I forsee many use cases of a good editor with SimBlog. Just go to [Atom](https://atom.io/), installing the exe and click to install. Easy and simple.

## Install git

I feel it is unnecessary to explain how necessary it is to have git on your computer once you start programming. Go to [Git Download page](https://git-scm.com/download/win), it will figure out which version you need and start downloading. You just click and install.

## Fork SimBlog and Configure Git page

You need your own repository for SimBlog and it will publish your posts on your Git pages. As the [setup.md](https://github.com/puneetsl/simblog/blob/master/setup.md) mentioned, [fork](https://github.com/puneetsl/simblog/fork) SimBlog to your own github account as the first step.

Now since this repository is your, you will want to start personalize your Git page with it. On project settings (the tab on the top right), give a new name to your repository (I chose "blog" as it suggested). On the same page, just scroll down to ```GitHub Pages``` and set master as your source branch (you could can later create a ```gh-pages``` branch and select it as your source branch). Now select your favorite theme from the drop-down for you blog. After this setting, you should see a banner with the link to your newly enabled Github blog page (mine is https://eunicechen.github.io/blog/). There were already some example posts created for you.

## Install Python

Before you clone this repository locally, you will want to have python and pip installed on your PC. I first tried to install python with Windows store as [Get started using Python on Windows for beginners](https://docs.microsoft.com/en-us/windows/python/beginners#:~:targetText=Install%20Python,-To%20install%20Python&targetText=Go%20to%20your%20Start%20menu,Select%20Get.) suggested but my install of python from the store stuck at install and errored out. However, downloading python with Python 3 Installer worked for me later. Here is the post with all the details [Insall Python](https://realpython.com/installing-python/#step-1-download-the-python-3-installer). The installation with python installer went pretty smoothly, the only trick was to tick **Add Python to path** before installing so you don't have to manually do that. This will also install pip at the same time so you don't have to worry about it.

## Clone the Repository and Configurations

Now that you have both git and pip installed, you can start cloning the repository and configure SimBlog locally. Follow [setup.md](https://github.com/puneetsl/simblog/blob/master/setup.md), run this under your desired folder

```
pip install -r _scripts/requirements.txt
```

As I have installed Atom at the beginning, apm is already installed on your computer. You could run on the blog folder with the following command:

```
apm install --packages-file atom-package-list.txt
```

If you don't want to deal with apm, the doc suggested you to install all the packages within the atom-package-list.txt in atom.

>press CTRL+SHIFT+P in atom and type install packages and themes and install following packages one by one
>```
busy-signal
date
intentions
language-markdown
linter
linter-markdown
linter-ui-default
markdown-image-paste
markdown-pdf
markdown-preview-enhanced
markdown-toc
markdown-writer
platformio-ide-terminal
script
tidy-markdown
tool-bar
tool-bar-markdown-writer
```

Now when you **double-click** your md file within atom it will show a preview on the right.

## A Small Fix on ```new_post.py``` Script

As the [setup.md](https://github.com/puneetsl/simblog/blob/master/setup.md) suggested, I should be able to start a new post by running the ```new_post.py``` script but I had a problem building it with a complain on line 9, ```replace``` is not a valid method for int. Line 9 looks like This

```python
    post_title = post_title.replace(" ", "-")
```

The variable came from

```python
event, (post_title,) = sg.Window('Insert your post title'). Layout([[sg.Text('Post title')], [sg.Input()], [sg.OK(), sg.Cancel()] ]).Read()
```

I am not familiar with PySimpleGUI and not even that much with python, but I printed the above code and it was

```
('OK', {0: 'Test'})
```

```post_title``` was casted the key of the first tuple in the dictionary ```{0: 'Test'}```. I didn't find a good way to simply give it the value of the tuple to post_title. So I did

```python
event, values = sg.Window('Insert your post title'). Layout([[sg.Text('Post title')], [sg.Input()], [sg.OK(), sg.Cancel()] ]).Read()
post_title = values.get(0)
```

Now it seems to work and created me this new post.

## (Unrelated) Windows Console

I personally don't really like the windows CMD console, and I use iterm2 on Mac at work. I was trying to find a good console app for Windows. I didn't spend too much time to search for all the possible solution. Some people suggested [Windows PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/getting-started/getting-started-with-windows-powershell?view=powershell-6) which has already came with my computer but I still don't like it. I was just obsessed with the look of [cmder](https://cmder.net/) plus it has the copy/paste option on the console.

<div class="embed"><iframe src="https://cmder.net/" frameborder="0" allowfullscreen></iframe></div>

I downloaded the full version of cmder. You can unzip the zip file wherever you want and execute the exe file. However, if you want to start it from a different console (how bored I am to do this), you could put your cmder folder to somewhere you like and add it to the system path. I put the folder on ```"C:\Users\<username>\AppData\Local\Programs\cmder``` then set this folder to be a system path. This answer on Stack Overflow gives a walkthrough on how to [Put a path into system path](https://stackoverflow.com/a/41895179).

I personally like to have a quick finder on my computer, just like you do ```cmd```+ ```space``` on Mac and search for whatever you want. I would like it a lot to open up my console by typing ```cmder``` on the start menu. I don't know a different way to do it, but I succeeded by adding a short cut of cmder.exe to Desktop, and then it is searchable from the windows start menu search bar from then on (it won't work once you remove the shortcut from desktop).
