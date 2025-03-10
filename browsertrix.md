---
layout: default
permalink: browsertrix
title: Browsertrix
---

## Web Crawling 

Web crawling is the process of systematically browsing a website or set of websites. Browsertrix is the tool SUCHO is using to crawl entire sites and copy all their contents for the purposes of emulation and replay. Most websites can be preserved in their entirety using this tool.

However, some websites have content (e.g. interactive 3D models) that won't capture well with Browsertrix; in these cases, we will send those parts of the websites over to be scraped/captured by other tools. 

Browsertrix is a little complicated to set up, so it has become a bottleneck for SUCHO. These instructions are written to help onboard people who are "medium-technical" (not necessarily people who code, but who aren't afraid of installing and messing around with stuff). Even if you have little prior coding experience, this tutorial should make it possible for you to crawl and preserve a website. 

By the end of a crawl, you will have generated a zipped file of a a series of [Web ARChive (WARC) files](https://en.wikipedia.org/wiki/Web_ARChive). This final zipped file produced by Browsertrix is known as a .wacz file (short for [Web Archive Collection Zipped](https://webrecorder.net/2021/01/18/wacz-format-1-0.html)). 

**If you have questions**, don't hesitate to ask on the #browsertrix Slack channel. This sort of work often requires help for troubleshooting.

### Browsertrix
[Browsertrix](https://github.com/webrecorder/browsertrix-crawler) is a simplified browser and crawling system that can create web archive files for entire sites. It's distributed as a *Docker container*. 

A [Docker](https://www.docker.com/) container basically packages up system configuration in a way that makes a software program easy to share and run on different computers and servers.

## Initial Set Up

### Installing Docker

The first step is to [download and install Docker](https://docs.docker.com/get-docker/). That link has instructions and download information for Mac, Windows, and Linux. 

*For Macs*: there are different links for Mac with Intel vs. Apple chip; most Macs have an Intel chip. For installation on Mac, you will need to ok security warnings confirming you intended to open and install the software, and you may also need to give Docker privileged access to install networking components.

*For Windows*: Installation should be pretty straightforward after downloading the executable file. If you run into trouble with the next steps, try restarting your computer after installation is complete.

Once Docker is installed, as it loads you should see a sort of whale-with-flickering-boxes in your computer's toolbar menu. This is visible on the top of the screen on Mac, bottom of the screen on Windows. You can minimize or close the Docker Desktop window, but you should still see a whale-with-boxes icon (a ceteceous shipping container). 

If you want to speed up Docker, you can look at advanced options to change how it uses computing resources. On Mac, go to 'Settings,' 'Resources,' and increase CPU usage, Memory, and other features. On Windows, these edits need to be made to the .wslconfig file (see their [docs](https://docs.microsoft.com/en-us/windows/wsl/wsl-config)).

### Launching the command line
Now that Docker is running, we can set up the web crawler from the command line.

*For Macs*: go to *Applications > Utilities > Terminal*.

*For Windows*: search for *cmd*, and the Command Prompt app should appear as the best match.

### Getting the Docker image for Browsertrix

In your command line, type or paste this:
`docker pull webrecorder/browsertrix-crawler`

This command downloads and sets up Browsertrix using Docker.

**Note**: If this command throws an error, you might not have administrative permissions. Try the above command again, but put `sudo` at the front, so the command would be: `sudo docker pull webrecorder/browsertrix-crawler`

Now that you've installed Docker and configured the Docker image, you shouldn't need to redo these first setup steps again. 

## Picking a website from the spreadsheet
Before you click on a link in the spreadsheet and open it in your browser, please read out [security guidelines](https://www.sucho.org/security).

Go to the Browsertrix tab of the SUCHO working spreadsheet and pick a site to work on that no one has claimed yet. To claim the site, on that row of the spreadsheet, add your name to the 'Claimed By' column, and update the 'Status' column to 'in progress.' 

Prioritize sites with links ending in `.ua`. Check where they are hosted and focus on sites in Ukraine and environs using [Hosting Checker](icehttps://hostingchecker.com).

Load the 'Collection Url' in your browser to see if it's working; many sites are already going down, so double check before proceeding. 

Next, to avoid downloading malware, please make sure your personal computer is backed up, and run it through a [security check](https://sitecheck.sucuri.net/). 

If the security check can't run on the site, or if the security risk appears to be severe, make a note in the Comments field and move on to the next item. We can assign other people to run the crawler on dodgier links using stand-alone servers. As far as we can tell, a "Medium" risk shouldn't pose a threat to you if the security check returns that no "malware" or "injected spam" is detected in the site.

## Creating a configuration YAML file 
A YAML file is a plain-text file for storing configuration information about how a programming script will run. YAML files are very picky about spaces, how many, and where they're located. Each time you conduct a crawl, you can edit a yaml file to configure the crawl for a website and its subdomains.

You can download an [example `crawl-config.yaml` file here](crawl-config.yaml), and modify it using a plain-text editor. (If you don't have a plain-text editor already installed on your computer, download and install [Atom](https://atom.io/) for Mac or Windows, and use that to open and edit the example YAML file.)

The `crawl-config.yaml` file should look as follows (with `collection`, `url`, and `include` changed to match each website): 

```
collection: "sgiaz-uamuseum-com"
workers: 8
saveState: always
seeds:
  - url: http://sgiaz.uamuseum.com/
    include: 
      - ^(http|https):.*sgiaz\.uamuseum\.com
    scopeType: "host"
```

Here's the fields you should modify each time:

* `collection:` this should be basically the URL that you scrape, but with hyphens instead of periods in the URL. So *http://archangel.kiev.ua* becomes `collection: archangel-kiev-ua`
* `url:` this is just the base URL in the SUCHO spreadsheet for the URL you're scraping
* `include:` this is a little tricky, but all you need to do is reconfigure the collection URL with some new syntax to ensure the webcrawler captures subdomains. It starts with `.*\.` and then the first part of your URL. Instead of a dot between the parts of your URL path, it should be `\.` So *http://archangel.kiev.ua* becomes `include: .*\.archangel\.kiev\.ua/`

Save the YAML file as `crawl-config.yaml` somewhere easy to navigate to on your computer -- on a Mac, the Documents folder is a good one. You will need to be able to change your directory using the command line to where your *crawl-config.yaml* file is saved on your computer to run the Docker command from that directory when you crawl the site. 

For examples of `crawl-config.yaml` files used for the SUCHO project, see our separate Github repository, [browsertrix-yaml-examples](https://github.com/sucho-archiving/browsertrix-yaml-examples).

## Starting to crawl the site
Open up the command line again, if you closed it before. 

*For Mac*: this will by default put you in your home directory (i.e. /Users/your-user-name). If you saved your *crawl-config.yaml* in the Documents folder, type `cd Documents`, and your command line will put you in the Documents folder. (If you put it somewhere else, you can put in that path after the `cd`, e.g. `cd Documents/some-subfolder/another-subfolder`).

Once you're in the same location as your *crawl-config.yaml*, paste this command into the Mac terminal and press enter to start the crawling:

`docker run -v $PWD/crawl-config.yaml:/app/crawl-config.yaml -v $PWD/crawls:/crawls/ webrecorder/browsertrix-crawler crawl --config /app/crawl-config.yaml --text --generateWACZ`

*For Windows*: after navigating to the right directory in the command prompt using `cd`, type the following command:
`docker run -v %cd%/crawl-config.yaml:/app/crawl-config.yaml -v %cd%/crawls:/crawls/ webrecorder/browsertrix-crawler crawl --config /app/crawl-config.yaml --text --generateWACZ`

### Troubleshooting the crawl command
You may have to use 'sudo' at the start of this command. 

At this stage, if you encounter errors relating to absolute paths, directories, or other errors, you may need to double check where you placed your config file, and how you are directing browsertrix to find it.

Some users on both Macs and Windows have had problems with $PWD and %cd%. If that doesn't work, put in the full system path to the crawl-config.yaml. Try putting the full path in quotes. 

On Windows, to find the absolute path for your .yaml file, locate the crawl-config.yaml file and copy the directory address in the folder window.

## Waiting
Depending on the size of the site, the crawl could take anywhere from a couple minutes to 10+ hours. If you run out of space on your computer, contact @Seb on the SUCHO Slack and he'll use one of the big servers on it.

### Interruptions
If the crawl gets interrupted, or you need to interrupt the execution in the command line, browsertrix should be able pick up where it left off if you run a slightly different crawl command. 

This is an example you would need to modify for your case: `docker run -v $PWD/crawls/collections/history-org-ua/crawls/crawl-20220305161714-00e289c2da70.yaml:/app/crawl-config.yaml -v $PWD/crawls:/crawls/ webrecorder/browsertrix-crawler crawl --config /app/crawl-config.yaml --generateWACZ --text --timeout 120`

The first argument now points to crawls/collections/....../crawl-[LOTSOFNUMBERS].yaml

If the crawl fails for any number of reasons, change the status to Failed and add notes about the errors and problems in the Comments field. Another person can try recrawling the site later with more complex parameters, or we may turn it over to manual webrecording tools. 

### Timeouts
If webpages fail to load and timeout, you may need to manually set browsertrix to a longer timeout limit by adding to the end of your command `--timeout 300`. Timeouts are tricky, so if you can't get it working, make a comment and move on to another open item. 

## Final Step: Uploading the WACZ file
The directory that has your *crawl-config.yaml* file will generate a *crawls* directory the first time you run the command to crawl a site. To find the WACZ file containg the archive of the website, open the  *crawls* folder, then the *collections* folder. Inside *collections*, you should see a folder for each collection you've crawled. Inside the collection folder is a .wacz file.

Verify the website was captured by uploading the .wacz file to the Webrecorder's [ReplayWeb.Page](https://replayweb.page/). Once the archival file is loaded into ReplayWeb.page, it is served locally on your machine, and you can navigate the website. Focus on verifying that the main subcomponents of the site were saved, especially pages listed in the navbar. Many links on the site may be external to the domain you preserved. 

Upload that .wacz file to our [WACZ uploads form](https://forms.gle/N18MxWgoHtPB2xpz8). Make sure to add info the Notes field about any errors you encountered and any concerns you have aboaut the quality of the .wacz file. The Quality Control team can verify your lingering questions. 

Once you've submitted the Google Form, you're crawl is complete! Thank you for your work. 

Please mark in the spreadsheet the row's status as "Submitted," and continue on to the next item.
