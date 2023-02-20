---
title: "Restoring many old, deleted files from different commits in Git"
author: Seth Jensen
date: 2023-02-20
tags:
- git
- tips
---

I recently needed to solve an interesting problem: the project I was working on needed to restore about 50 images which had been deleted in commits scattered across a 15-year time span.






We used to cycle through certain files, deleting them as they became less relevant. But recently, we decided to keep all the old files for display in archival content.

If we had deleted all of these files in the same commit, there would be a few straightforward options for restoring them:

* Grep through git log to find the commit before they were deleted, then there either checkout the files from that commit directly.
* Detach the HEAD at that commit and copy the files to a `tmp` directory outside the repo.
* Use `git show` to retrive the raw data from each file's blob.

However, our files were deleted on a rolling schedule, not in one convenient commit. So restoring them was a little more involved:

### Git log output

First, we need a git log we can work with:

```plain
$ git log --format=format:"commit %H" --raw > git-log-output.txt
```

* We give the `--format` option a format string, since we only care about the commit hash, which we select with `%H`.
* `--raw` outputs file changes in the raw format, which will show where files were deleted, as well as the glob for the file, which we need for restoration.
* We can save processing time by redirecting this to a text file, so we don't need to run it every time we loop later.

### Fetching the files

For our example, we'll choose a few arbitrary sets of files, each captured with a regex:

1. Any banner image: `D\s*\S*banner\S*?\.(png|jpg|jpeg)`

    Here we check for a `D` to signify a deleted file, then any amount of whitespace, then any amount of non-whitespace containing "banner". That will catch "banner" in the filename or the parent directories.

2. Any `html` files in the `article` directory: `D\s*article\S*?\.html`
3. A single file called `liveblog-epic-design-test.js`


```bash
for i in "D\s*\S*banner\S*?\.(png|jpg) D\s*article\S*?\.html D\s*article\S*?\.html"; do

  current=$(sed -nE "/^commit|$i/p" git-log-output.txt | grep -m 1 -B 1 -iP "$i") # ain't gonna work if your regexes intend to capture more than one file

  if [[ ! -z $current ]] && [[ "$current" =~ 'rails' ]]; then
    path=$(echo $current | sed -nE "s/^(\S*\s*){4}([^ ]*)$/\2/p")
    git show $(echo $current | sed -nE "s/^commit ([^ ]*).*$/\1/p")^:$path > restored-images/$(echo $path | sed -nE 's/^.*\/(.*)$/\1/p')
  fi  

done
```

> I had enough different filenames that I put them in a file, each seperated by a space. Then I had my for loop iterate over `$(cat myfilenames.txt)`
