---
layout: post
title:  "How to Create and Post Gitpages"
date:   2015-05-23 13:47:01
categories: How to
---

*Please note, theese instructions are based on Ubuntu 14.04 as the host OS.

Sometimes it can be awhile between posts and if I get busy learning new things, the old info may get pushed out of recent memory.  This post is a quick-start guide so I don't forget how to make and push simple posts in the future.

### Installation & Setup

Prerequisites: Install the following prerequisite programs (Ruby, RubyGems, Jekyll)
	
	#Install Ruby
	sudo apt-get install ruby-full
	
	#Install Rubygems
	cd /opt
	sudo git clone https://github.com/rubygems/rubygems
	cd rubygems
	ruby setup.rb

	#Install Jekyll
	gem install jekyll

	#Install Node.js
	apt-get -y install curl
	curl -sL https://deb.nodesource.com/setup_0.12 | sudo bash -
	sudo apt-get -y install nodejs
	
Cloning this Gitpages Repo: Go to the folder you want to store the project, and clone the new repository:

	git clone https://github.com/ajthompson0659/ajthompson0659.github.io
	
Run Gitpages on localhost: From within the Gitpage repository's root directory, start the Jekyll server.  
This will allow you to view the pages locally (http://127.0.0.1:4000) as they will appear when pushed to the live site.
	
	#navigate to the repository's root directory
	cd ajthompson0659.github.io

	#start the Jekyll server in detached mode
	jekyll serve --detach

Using a web browser, navigate to the local repository URL http://127.0.0.1:4000 to view the site.

Set Git Account Default Identity:

	git config --global user.email "you@example.com"
	git config --global user.name "Your Name"
	git config --global push.default matching


### Creating Posts

Post Files: Post files can be created using any text editor and plain text.  Post files are files with the .markdown file extension that are stored in the _posts directory in the root of the Gitpage repository.  The file name must use the following naming convention: YEAR-MONTH-DAY-title.markdown.  See example below.

	nano /path-to-repository/_posts/YEAR-MONTH-DAY-title.markdown

Post File Headers:  All post files start with a header (aka YAML Font Matter) that begins and ends with a line of triple dashes (---) with required labels in on the lines in-between.  See example below.

	---
	layout: post
	title:  "How to Create and Post Gitpages"
	date:   2015-05-23 13:47:01
	categories: How to
	---

Headings: Create headings by placing pound (#) before the heading text.  Multiple # means bigger headings.  See example below.

	### Heading 1

Text and Paragraphs: Text appears as typed within the post.  

Hyper-links: Enclose the hyper-link text in brackets ([hyper-link text]), then immediatly follow the closing bracket with parenthasis ((hyper-link URL)) containing the hyper-link URL.  See example below.

	[Ubuntu 14.04 LTS Server 32-bit](http://releases.ubuntu.com/14.04/) 

Lists: Create bulleted lists by placing an asterik (*) before each list item.  See example below.

	* list item 1
	* list item 2
	* list item 3

Code Snippets: Display code snippets by indenting the code with a tab before each line of code.  See example below.

	apt-get -y install openvswitch-common openvswitch-controller openvswitch-dbg \
	openvswitch-ipsec openvswitch-pki openvswitch-switch openvswitch-datapath-source \
	openvswitch-test openvswitch-datapath-dkms

Pictures: Embed a picture by placing an exclamation point (!) before the hyper-link URL reference.  See example below.

	![My Network Lab Rack](/images/labrack-rear-view.jpg)

![My Network Lab Rack](/images/labrack-rear-view.jpg)

### Pushing Posts from the CLI

If you created posts locally on your computer, you can use git commands to push the posts to the live repository.  See below example.

	git add 2015-05-23-how-to-create-and-post-gitpages.markdown
	git commit -a -m "Commit Description"
	git push origin 
	
Go to your repository's live site URL and view the changes (http://username.github.io).  To see the changes locally, kill and restart the Jekyll serve process.

For further reading, look at [GitHub Pages Basics](https://help.github.com/categories/github-pages-basics/).

