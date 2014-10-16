---
layout: post
title: "How to set up Octopress and push to github pages"
date: 2014-10-11 02:45:46 -0700
comments: true
categories: [octopress, git]
---

This is my first time setting up a blog using Octopress/Jekyll and hosting it on Github Pages. I wanted to document the process so I could remember it and also share it with others. Any comments or corrections are welcome. 

<!--more-->

## Initial Setup

Please note that I'm working off a Macbook Pro with OSX 10.9.5.

1. Get a [github account](https://www.github.com)
2. Get [Git](http://git-scm.com/downloads)
3. [Setup Git](https://help.github.com/articles/set-up-git/) and learn how to [fork a repo](https://help.github.com/articles/fork-a-repo/)
4. Get Ruby 1.9.3 or higher using [rbenv](https://github.com/sstephenson/rbenv) or [RVM](http://rvm.io/rvm/install)
5. Run `ruby --version` in your [terminal](http://blog.teamtreehouse.com/introduction-to-the-mac-os-x-command-line) to make sure you're running at v1.9.3 or higher

## Setup Octopress 

1. Login to github and go to [Octopress](https://github.com/imathis/octopress)
2. At the top right of Octopress' [repo page], fork the repo by click on the [Fork](https://github.com/imathis/octopress/fork) button at the top right
3. You should have a fork of Octopress in your own repo, like I do at [https://github.com/jonathanxie/octopress](https://github.com/jonathanxie/octopress)
4. Clone your own fork to your `Desktop` by using [Github's desktop client](github-mac://openRepo/https://github.com/jonathanxie/octopress) or with the following commands in terminal: `git clone https://github.com/YOUR_GIT_USERNAME/octopress.git` - Make sure to use your git username here.
5. Go to cloned repo you just created: `cd octopress`
6. Download Octostrap3 theme into a hidden directy called .themes:<br/>`git clone https://github.com/kAworu/octostrap3 .themes/octostrap3`
7. Install the Octostrap 3 theme: `rake 'install[octostrap3]'`
8. Generate the pages: `rake generate`
9. Test drive the blog by starting the built in web server: `rake preview watch` 
10. Go to your browser such as Google Chrome and go to: `http://localhost:4000`

<br/>
Here are the steps 4-9 that you would execute line by line in your [terminal](http://blog.teamtreehouse.com/introduction-to-the-mac-os-x-command-line):

<script src="https://gist.github.com/jonathanxie/25ebaa61034fbe9c2dc0.js"></script>

In terminal, `push CTRL-C` to stop the web server.

## Git and Github

Now I will explain some things about git. Earlier you created a fork of Octopress to your github account. My fork is at:

[https://github.com/jonathanxie/octopress.git](https://github.com/jonathanxie/octopress.git)

Your fork would be at:

ht<span/>tps://github.com/`YOUR_GITHUB_USERNAME`/octopress.git

Please take note that URL address is what git considers a `remote repository`. Then you cloned Octopress from this remote repository into a Desktop directory called `Octopress` by using Github's desktop application or with the following command:

```
cd ~/Desktop
git clone https://github.com/jonathanxie/octopress.git

```

### Origin and Remote Repositories

This tells git to create a local copy of your forked Octopress to your desktop at `~/Desktop/octopress`. One important thing to note is that git will set this `remote repository` as the `origin` for your local repository. When you commit your code from your local copy, all your changes will go to `origin`. `origin` points to the remote repository where you want to publish your commits. By default, the remote repository you cloned from is called "origin", but you can work with several remotes that even have different names concurrently. This is a **very important** concept you will need to understand in this post, but you can learn more about remote repositories [here](http://gitref.org/remotes/).

To see your remote repositories, in your terminal type: `git remote -v`

For the output, you should see something like:

{% codeblock Output for: git remote -v %}
origin  https://github.com/jonathanxie/octopress.git (fetch)
origin  https://github.com/jonathanxie/octopress.git (push)
{% endcodeblock %}


As you can see, your `origin` for fetching and pushing code points to the remote repository you forked earlier. 


### Branching

Another important thing to note here about git is `branching`. A git repository can have multiple branches and you use git branches to develop code isolated from each other. An example of how branching looks like is shown here taken from Atlassian's thorough branching [tutorial](http://localhost:4000/blog/2014/10/11/how-to-set-up-octopress-and-push-to-github-pages/): 

![Alt Atlassian's example of branching in git](https://www.atlassian.com/git/images/tutorials/collaborating/using-branches/hero.svg)

<br/>
Right now your local repository has one branch called `master`. To check what branch you are on, run the following command: `git branch`

You should see:

{% codeblock Output for: git branch %}
* master
{% endcodeblock %}

The `*` in front of `master` represents the current branch you are working on in the repository. So note that you are currently working in the `master` branch of your repository. When you run `git init` to create a local working repository or `git clone https://github.com/jonathanxie/octopress.git` to copy a remote repository, git will **automatically create a `master` branch for you by default to start off with in your repository**. 

Now say for example, you and I are working on the same remote repository but developing different features. I can create a branch from the master branch called `jonathan-feature-x` and switch to that new branch. Then you could stay on the master branch without worrying about my chances until we merge code. The code would look like:

{% codeblock Code to create and switch to a new branch %}
git branch jonathan_new_feature
git checkout jonathan_new_feature
{% endcodeblock %}

`git checkout` is the way to switch to a new branch. When you run that command you should see the following output:

{% codeblock Output for: git new branch %}
Switched to branch 'jonathan_new_feature'
{% endcodeblock %}

Note that I could have also done this in one line with: `git checkout -b jonathan_new_feature`

Now run `git branch` and you should see:

{% codeblock Code to create and switch to a new branch %}
* jonathan_new_feature
  master
{% endcodeblock %}

Note the `*` next to `jonathan_new_feature`. This means you are now working on `jonathan_new_feature` instead of `master`. To switch back, just run: `git checkout master` and you will see:

{% codeblock Output for: git checkout master %}
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
{% endcodeblock %}

The status message saying `Your branch is up-to-date with 'origin/master'.`  means that there are no merges or changes after switching branches.

You can now delete the newly created branch with: `git branch -D jonathan_new_feature`

You should see the following out:

{% codeblock Output for: git branch -D jonathan_new_feature %}
Deleted branch jonathan_new_feature (was a311b68).
{% endcodeblock %}

Run `git branch` again and you will see:

{% codeblock Output for: git branch %}
* master
{% endcodeblock %}


## Creating your first blog post on Octopress

Now that you some understanding about git and Github, it's time to create your first blog post. In terminal, run:

{% codeblock Code to create a new blog post %}
rake new_post["Hello World"]
{% endcodeblock %}

You should see the following output:

{% codeblock Output for: rake new_post["Hello World"]%}
mkdir -p source/_posts
Creating new post: source/_posts/2014-10-11-hello-world.markdown
{% endcodeblock %}

You can now edit the file using [markdown](https://help.github.com/articles/markdown-basics/) at the following directory:

`~/Desktop/octopress/source/_posts/2014-10-11-hello-world.markdown`

I prefer to use [Sublime Text](http://www.sublimetext.com/) and I've setup it up to use from [command line](https://www.sublimetext.com/docs/2/osx_command_line.html). You can use any text editor but if you have Sublime Text like me, run the following command:

{% codeblock Run Sublime Text %}
subl ~/Desktop/octopress
{% endcodeblock %}

Then push `Command-T` in Sublime Text and type in `2014-10-11-hello-world.markdown` in the file search dialog bog and open that file. You will then see `[front matter](http://jekyllrb.com/docs/frontmatter/)` text:

{% codeblock Front Matter at top of markdown file %}
---
layout: post
title: "Hello World"
date: 2014-10-13 16:04:07 -0700
comments: true
categories: 
---
{% endcodeblock %}

Add something simple below that: `Hello World! This is my first post!`

Save the file, then run: `rake preview watch` and go to: [http://localhost:4000](http://localhost:4000)

You will see a blog post called `Hello World` and the url should be: 

[http://localhost:4000/blog/2014/10/13/hello-world](http://localhost:4000/blog/2014/10/13/hello-world)


## Saving your code to Github

Now that you have your first blog post, it's time to save your it to your repository.

Run the following command: `git status`

You should see the following output:

{% codeblock Output for: git status %}
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

  source/_posts/2014-10-11-hello-world.markdown

nothing added to commit but untracked files present (use "git add" to track)

{% endcodeblock %}

So you're on the `master` branch with an untracked file: `source/_posts/2014-10-11-hello-world.markdown`

Run this command to add the file: `git add source/_posts/2014-10-11-hello-world.markdown`

Now the file is added and tracked by git. Next run: `git commit -m "Adding Hello World blog post"`

Then you should see:

{% codeblock Output for: git commit - m "Adding Hello World blog post" %}
[source c779ac2] Adding test file
 1 file changed, 7 insertions(+)
 create mode 100644 source/_posts/2014-10-14-hello-world.markdown
{% endcodeblock %}

Now your changes have been committed to your local copy of the repository but you have to push them to your remote repository. Remember you're on the `master` branch right now and your remote repository is called `origin`. **So you need to tell git to push your committed changes from the master branch of your local repository to the master branch of the remote repository at origin**

Run: `git push origin master`

You should see something like:

{% codeblock %}

Counting objects: 18, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 340 bytes | 0 bytes/s, done.
Total 4 (delta 3), reused 0 (delta 0)
To https://github.com/jonathanxie/octopress.git
   ca939a8..8c84805  master -> master

{% endcodeblock %}





### Setting up Octopress to deploy to Github Pages

Now that you have successfully pushed your code to github, it's time to setup your blog so that Github Pages can host it. The original tutorial on Octopress' site is [here](http://octopress.org/docs/deploying/github).

Github Pages allows for free hosting of your blog! So first go to your github account and create a repository called `username.github.io`, where `username` is your GitHub user name or organization name. Do not add an LICENSES or README files just yet when you set up the page. You will need a clean repository. 

My repository is called `jonathanxie.github.io` and you can go to https://jonathanxie.github.io to view my blog too. I'm able to use http://www.jonxie.com because I have a custom domain with Github Pages, which you can read about [here](http://octopress.org/docs/deploying/github/#custom_domains).

Now copy your repository's HTTPS url, it should looking something like:

{% codeblock Example of Github blog repository URL for Github Pages %}
https://github.com/jonathanxie/jonathanxie.github.io.git
{% endcodeblock %}

Go back to the terminal and run the following command to setup deployment to Github Pages:

{% codeblock Command to setup Octopress for Github Pages %}
rake setup_github_pages
{% endcodeblock %}

This rake task will:

1. Ask for and store your Github Pages repository url: `https://github.com/YOUR_GITHUB_USERNAME/YOUR_GITHUB_USERNAME.github.io.git`
2. Rename the remote respository pointing to `https://github.com/YOUR_GITHUB_USERNAME/octopress` from 'origin' to 'octopress'
3. Add your Github Pages repository pointing to `https://github.com/YOUR_GITHUB_USERNAME/YOUR_GITHUB_USERNAME.github.io.git` as the default origin remote.
4. Switch the active branch from `master` to `source`. 
5. Configure your blog's url according to your repository.
6. Create a directory called `_deploy` which contains your static code for your blog
7. Setup a master branch in this `_deploy` directory
8. Push the code in this new master branch to the new `origin`

Now if everything is setup correctly, you should be able to view your blog at: 

http://YOUR_GITHUB_USERNAME.github.io



### Branches and origins have changed

Note that your branch has changed from `master` to `source` as stated by step 4. Run: `git branch`

{% codeblock Output for: git branch %}
* source
{% endcodeblock %}

Also note that your remote repositories have changed as stated in Steps 2 and 7. Run: `git remote -v`

{% codeblock Output for: git remote -v %}
octopress https://github.com/jonathanxie/octopress.git (fetch)
octopress https://github.com/jonathanxie/octopress.git (push)
origin  https://github.com/jonathanxie/jonathanxie.github.io (fetch)
origin  https://github.com/jonathanxie/jonathanxie.github.io (push)
{% endcodeblock %}


## rake deploy

**IMPORTANT NOTE:** Make sure to always run `rake generate` before `rake deploy`. If not, your changes in your code will **NEVER** get pushed to Github Pages. 
<br/>
<br/>
Now when you run `rake deploy`, your code from the `source` directory will be compiled to the `_deploy` directory, which has `master` branch. `rake deploy` will also commit and push the code from the `_deploy` directory to the remote repository at `origin`: https://github.com/jonathanxie/jonathanxie.github.io

So you shouldn't have to ever go into the `_deploy` directory to do anything. 



## Saving your code to your Octopress remote repository.

It is important to note that `rake deploy` does not save your code to your Octopress remote repository. You still need to save the code you typed up until now. To commit and push your code in the `source` directory to your Octopress repository at https://github.com/jonathanxie/octopress.git, run: 

{% codeblock Code to commit and push your code to the Octopress remote repository %}
git add -A 
git commit -m "Add some all modified and deleted files"
git push octopress source
{% endcodeblock %}

You should see something like:

{% codeblock Output for: git push octopress source %}
Counting objects: 20, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 1.48 KiB | 0 bytes/s, done.
Total 5 (delta 4), reused 0 (delta 0)
To https://github.com/jonathanxie/octopress.git
   8c84805..524526c  source -> source
{% endcodeblock %}

`git push octopress source` means you are pushing from the `source` branch to `octopress`, which is a remote repository pointing to: https://github.com/YOUR_GITHUB_USERNAME/octopress.git

When you push from `source` to `octopress`, git will create a new branch in the remote repository on Github. So if you need to clone your repository again, you should use the following command (remember to change `jonathanxie` to **your github username**):

{% codeblock Command to clone the 'source' branch of your Octopress remote repository %}
git clone https://github.com/jonathanxie/octopress.git -b source --single-branch
{% endcodeblock %}

## References

### Octopress 
http://octopress.org/docs/blogging/ 
<br/>http://octopress.org/docs/deploying/github/
<br/>http://allenyee.me/blog/2013/08/21/what-i-learned-from-hosting-octopress-on-github/
<br/>http://learnaholic.me/2012/10/10/deploying-octopress-to-github-pages-and-setting-custom-subdomain-cname/
<br/>http://webdesign.tutsplus.com/tutorials/getting-started-with-octopress--webdesign-11442
<br/>
<br/>
### Git Guides
https://help.github.com/articles/set-up-git/
<br/>http://rogerdudler.github.io/git-guide/
<br/>http://gitref.org/remotes/
<br/>https://www.atlassian.com/git/tutorials/using-branches/
<br/>http://gitref.org/branching/#branch
<br/>http://stackoverflow.com/questions/9529497/what-is-origin-in-git
<br/>
<br/>
### Markdown for blogging
https://guides.github.com/features/mastering-markdown/
<br/>http://markdowntutorial.com/
<br/>https://daringfireball.net/projects/markdown/basics
<br/>http://mathewmitchell.net/tutorials/markdown/