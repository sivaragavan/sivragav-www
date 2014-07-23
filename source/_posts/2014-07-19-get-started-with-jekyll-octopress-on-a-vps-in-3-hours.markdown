---
layout: post
title: Get Started with Jekyll/Octopress on a VPS in 3 Hours
date: 2014-07-19 11:03:53 +0530
comments: true
categories: writing jekyll octopress digitalocean vps blogging git markdown
---

Over the past few months, I have been exploring different options to host/publish this blog. After trying [Medium][1], [SquareSpace][2], [Tumblr][3] and [Wordpress][4], I finally settled with [Jekyll/Octopress][5]. Even though the setup time is a bit long, it is worth it. The control you get is simply not comparable with other platforms. [Jekyll][6] is just a tool that generates static HTML sites from a directory of Markdown files. [Octopress][7] is a framework designed for Jekyll, to cut down some steps from the initial setup phase.

The setup wasn’t a smooth ride. I hit certain bumps here and there. But managed to get this awesome site up and running in less than 3 hours. If you are thinking about hosting your own Jekyll/Octopress blog, Read on.

<!--more-->

## Hosting Options
This is the first thing you need to decide when you choose a open source blogging platform like Jekyll. The author of Octopress, Brandon Mathis, has listed down the different hosting options [here][8]. All you need is a webserver. Here are my takes on each of those options :

* [GitHub pages][9] - This is the default option most of them opt for. GitHub provides free hosting for one repository for every user/project/organization. All you need is a repo named “username.github.com” and you can push your jekyll generated site into it. Even though GitHub can generate the site for you everytime you checkin a markdown file, it wouldn’t work with octopress or any other plugins. GitHub forces safe-mode while generating the site. This disables all plugins. So GitHub pages is nothing more than a web server, when it comes to Jekyll/Octopress.
* [Heroku][10] - You can host your site as a dyno in heroku. It will detect your app as a Ruby-on-Rails app. The downside of using Heroku is that, there is no easy way to add a post-receive hook for heroku’s Git repositories. So you can’t put up a script that can generate your site when you push a markdown file. This means, you will have to manually generate the site in your local setup and push the generated site to heroku. So Heroku is also nothing more than a webserver in this case.
* [Virtual Private Server][11] - VPS ([Digitial Ocean][12] / [Linode][13] / …) - I finally settled for a VPS that can be tweaked for any kind of usage. I decided to setup a Git Server and push my Jekyll/Octopress source (not the generated site) to this server. I was able to setup a post-receive hook that can generate the site after every push. This would place the generated static site in a particular location where Nginx can serve it from. To post an article, all I have to do is to push the markdown file to my Git Server and I am all set. My site is updated within seconds. Unlike the other approaches, where I had to generate the site after every new post and push the changes in the generated html files. This keeps my repo clean with only the source files and no generated html files. It also doesn’t need to be manually generated everytime, I want to post an article.

Let’s take a look at the steps involved in the thrid approach. This [blog post][14] from Digital Ocean covers some of the steps in this process (for a jekyll only site). But it doesn’t cover octopress.

## Local Setup : Creating your Jekyll/Octopress blog

### Install dependecies (if missing)

Ruby :

This command installs Ruby 2.0 using RVM

{% codeblock %}
curl -L https://get.rvm.io | bash -s stable --ruby=2.0.0
{% endcodeblock %}

Git :

Use your favorite package manager to install git. In Ubuntu, that would look like :

{% codeblock %} 
apt-get install git
{% endcodeblock %}

### Setup Octopress

{% codeblock %} 
git clone git://github.com/imathis/octopress.git octopress
mv octopress my-blog
cd my-blog
{% endcodeblock %} 

Install Jekyll and its dependencies

{% codeblock %} 
gem install bundler
bundle install
{% endcodeblock %} 

Install the default theme

{% codeblock %} 
rake install
{% endcodeblock %} 

Preview your Blog

{% codeblock %} 
rake preview
{% endcodeblock %} 

Your blog is live now. It should be availble @ http://localhost:4000

### Optional Steps

If you want to use a different theme : [3rd Party Octopress Themes][15]

If you want to configure your blog : [Start Here][16]

## Server (VPS) Setup : Setting up Nginx, Git Server and Jekyll/Octopress
I decided to go with Ubuntu 14.04 VPS on [Digital Ocean][17]. [Linode][18] was the closest second option. It completely depends on your preferences. Take a look at both the services and decide for yourself. In any case, the below steps should work. Some of the below steps are specific to Ubuntu. If you are using a different distro, refer to the corresponding documentations.

> Use coupon **“SHIPITFAST10”** for a **10$** Credit in Digital Ocean. It can host your site free for 2 months.

### Non-Root User
Setup a non-root user on your VPS. You can follow the document [here][19], If you are unfamiliar with it.

### Setup Nginx
Nginx is a high performance web server and is highly customizable. For now, all you need is a plain web server and Nginx should do. Use your favorite package manager or tar ball to [install Nginx][20]. In Ubuntu, it should look like this.

{% codeblock %} 
sudo apt-get install nginx
{% endcodeblock %} 

If you are using a fresh VPS, you might want to use this command to refresh your packages in apt-get

{% codeblock %} 
sudo apt-get update
{% endcodeblock %} 

Try visiting http://domain\_name\_or\_IP. You should see the default Nginx landing page. Let’s assume your public HTML folder is /usr/share/nginx/html (which is the case in ubuntu 14.04 + latest Nginx), though it may be different depending on your distro and configuration. Make a note of this Nginx public folder. You will need it later.

### Setup Git Server
In Ubuntu, this should work.

{% codeblock %} 
sudo apt-get install git-core
{% endcodeblock %} 

Create an empty Git repo to push your blog:

{% codeblock %} 
cd ~/
mkdir repos && cd repos
mkdir my-blog.git && cd my-blog.git
git init --bare
{% endcodeblock %} 

### Setup Jekyll/Octopress
You need to repeat some of the steps from your local setup to install Jekyll/Octopress in this server.

Ruby :

This command installs Ruby 2.0 using RVM

{% codeblock %} 
curl -L https://get.rvm.io | bash -s stable --ruby=2.0.0
{% endcodeblock %} 

Setup a temporary Octopress repo :

{% codeblock %} 
cd ~/
git clone git://github.com/imathis/octopress.git octopress
cd octopress
{% endcodeblock %} 

Install Jekyll/Octopress and its dependencies :

{% codeblock %} 
gem install bundler
bundle install
{% endcodeblock %} 

Remove the temp repository :

{% codeblock %} 
cd ~/
rm -R octopress
{% endcodeblock %} 

## Workflow Setup : Pushing to Server and Post-receive hook
All the necessary setup is done. Let’s hook all of these together.

### Setup Post-receive Hook

Add a post-receive hook to your Git repo in your server. This is a shell script that runs everytime something is pushed to the server.

{% codeblock %} 
cd ~/repos/my-blog/
cd hooks
touch post-receive
vim post-receive
{% endcodeblock %} 

Copy the below lines of code into post-receive file. Adjust the variables accordingly. Remember the Nginx public folder you noted down earlier. Make sure the PUBLIC\_WWW in the below script points to this folder.

{% codeblock %} 
#!/bin/bash -l
GIT_REPO=$HOME/repos/my-blog.git
TMP_GIT_CLONE=$HOME/tmp/git/my-blog
PUBLIC_WWW=/usr/share/nginx/html
 
git clone $GIT_REPO $TMP_GIT_CLONE 
cd $TMP_GIT_CLONE
rake generate
cp -R public/* $PUBLIC_WWW/
cd -
rm -Rf $TMP_GIT_CLONE
exit
{% endcodeblock %} 

Let’s save this file and give it exec permissions.

{% codeblock %} 
chmod +x post-receive
{% endcodeblock %} 

### Setting up Git and Git Remote in Local setup

To push to a Git Server, you need to make your blog a git repo and add a remote pointing to your server. Navigate to my-blog folder in your local setup and follow these commands.

{% codeblock %} 
git init
git add .
git commit -m "Initial commit”
git remote add server user@domain_name_or_IP:repos/my-blog.git
git push server master
{% endcodeblock %} 

Now your repository is pushed to your server.

## Start Blogging

You are all set now. Let’s write the first blog post. Use the below command to create a new post in my-blog folder in your local setup

{% codeblock %} 
rake new_post[”my_blog_post_title”]
rake preview
{% endcodeblock %} 

Head over to http://localhost:4000 . You can see your blog with this new post without any contents in it. Edit it the way you like it and Have fun blogging.

Remember to push the changes to the server when you are done.

{% codeblock %} 
git push server master
{% endcodeblock %} 
 
This command will push the new post to your server and execute the post-receive hook. Once the script is done executing, you should be able to see your blog live on your server.

Try visiting http://domain\_name\_or\_IP.

You can find more details about blogging in Octopress [here][21].

Post your Questions/Corrections/Suggestions to the above approach in comments section below.


[1]:	http://medium.com
[2]:	http://squarespace.com
[3]:	http://tumblr.com
[4]:	http://wordpress.com
[5]:	http://octopress.org/docs/setup/
[6]:	http://jekyllrb.com
[7]:	http://octopress.org
[8]:	http://octopress.org/docs/deploying/
[9]:	http://pages.github.com/
[10]:	http://heroku.com
[11]:	http://en.wikipedia.org/wiki/Virtual_private_server
[12]:	https://www.digitalocean.com
[13]:	https://www.linode.com
[14]:	https://www.digitalocean.com/community/tutorials/how-to-deploy-jekyll-blogs-with-git
[15]:	https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes
[16]:	http://octopress.org/docs/configuring/
[17]:	http://do.co
[18]:	http://linode.com
[19]:	https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04
[20]:	http://wiki.nginx.org/Install
[21]:	http://octopress.org/docs/blogging/