# Hugo basics


## Create a new site in a directory <site_name>
``` sh
hugo new site <site_name>
```

## Running the server locally (-D flag is to render pages marked as drafts)

``` sh
hugo server -D
```

## Github

This repo has the alternate stream set to 'development'. So pushing master to
here will not effect the site itself.

## Deploying

``` sh
./deploy.sh
```
This script deploys the website to the repo georgesims21.github.io as a proper
site. It will not render files marked as drafts, so it should look exactly the
same as locally if you run:
```sh 
hugo server
```

## Structure

## Archetypes/
An 'archetype' is something common across the whole website. Can be
metadata, or data about data found in here.
    
The default.md file contains the frontmatter defaults for each newly created page.
You can create <directory_name>.md to make specific directory archetypes.
## Content

  - Contains all of your pages (content of the website)

### Organisation

Within content/ you can create singular pages and directories containing pages
for better organization. 
structure/dir1/pageA.md would have a URL: <localhost:1313> or <github.io>/dir1/pageA

The homepage will contain all single page content in the folders. If you go to
<>/dir1, you will then only see files within the dir1 directory. This only goes
1 level deep, so <>/dir1 will show files but <>/dir1/dir2 wouldn't show a list
page for dir2, you must create it manually by adding an index file to dir2.

### Index pages

``` sh
hugo new content/dir1/dir2/_index.md
```
Index pages are used to put information on the list pages (directories). If a
dir doesn't have one then you cannot write custom content. They must be named
'_index.md' to work, but they can have different titles within the index
metadata.

### Frontmatter/metadata

Located at the top of each page/index and contains key:value pairs. Can be
written in YAML(default), TOML and JSON. Most of this data will be displayed on
list pages, i.e if you have a title : "A" and date : "12th Apr" on a content file, then the blog
post will have this info when on the coresponding list page.

You can define custom frontmatter variables to use on each page.

#### Taxonomies

Ways to logically group content on site, tags and categories. Defined by:

``` json
tags: ["tag1", "tag2", ...]
categories: ["cat1", "cat2", ...]
```
HUGO automatically creates list pages in tags/<tagname> and links tags together.
If you want to define a custom taxonomy properly, you can define it in the same
way as tags/categories but inside the config.toml file you must add it.

``` toml
<default inside file>
... 

[taxonomies]
  tag = "tag"
  category = "categories"
  mood = "moods"
```
By default tags/cats work out of the box, but whenever you want to define a
custom one you must add them manually like above plus the new one.
A server restart is needed for the changes to take effect.


## Data
  - Acts like a database in a way. Can have data files you wish to pull info
    from on your site
## Layouts
  - Things like: I want to have the same header and footer on each page. This
    can be defined within files in here
## Static
  - Storing all the static elements of the website. Things that don't change
    (image, css file etc)
## Themes
  - Contains a downloaded theme. Should run
  
``` sh
git submodule add <theme_repo> themes/<name>
```

## Shortcodes (auto-html/templates)

To avoid working with html to add things like embedded videos from YT or
similar, shortcodes can be used to do this automatically.

``` toml
{{/< <shortcode_name> <param1> <param2> ... />}} # Generic
{{/< youtube <video_id> />}} # To embed YT vid
```



