# briancain.net

Go to www.briancain.net to view the website.

## Setup

```
bundle install
jekyll serve -w
```

## Build

```
jekyll
zip -r site.zip _site
mv site.zip files/
git add files
git commit -am 'Update build artifact'
```
