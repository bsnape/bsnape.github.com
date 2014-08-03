---
layout: post
title: Converting a Mercurial Repository to Git
---

Today I had the quick task of converting an old, internally-hosted Mercurial repository to Git - and subsequently pushing the converted repository to GitHub.

It was a lot easier than I thought it would be! Hopefully this should help others as it appears there are a number of ways to do this.

### The Conversion

First of all, clone the repository you wish to convert.

{% highlight bash %}
$ hg clone http://mercurial/old-repository
{% endhighlight %}

Now create a directory with the name of your project in a separate location.

{% highlight bash %}
$ mkdir converted-project
$ cd converted-project
{% endhighlight %}

Next, we need to clone the <a href="">Mercurial-Git converter</a> into the directory we just created ('converted-project').

{% highlight bash %}
$ git clone git@github.com:frej/fast-export.git .
{% endhighlight %}

Once that's done, we should remove the existing Git references in our directory before re-initialising it.

{% highlight bash %}
$ rm -rf .git .gitignore
.git/       .gitignore

$ git init
Initialized empty Git repository in /Users/bensnape/converted-project/.git/
{% endhighlight %}

We're ready to do the actual conversion now. We simply point the bash script <code>hg-fast-import.sh</code> to our Mercurial repository that requires converting with the <code>-r</code> flag. You should see some lengthy output from the conversion.

{% highlight bash %}
$ ./hg-fast-export.sh -r ../converted-project/

master: Exporting full revision 1/38 with 8176/0/0 added/changed/removed files
Exported 1000/8176 files
Exported 2000/8176 files
Exported 3000/8176 files
Exported 4000/8176 files
Exported 5000/8176 files
Exported 6000/8176 files
Exported 7000/8176 files
Exported 8000/8176 files
Exported 8176/8176 files
master: Exporting simple delta revision 2/38 with 0/2/0 added/changed/removed files
master: Exporting simple delta revision 3/38 with 0/84/0 added/changed/removed files
master: Exporting simple delta revision 4/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 5/38 with 0/0/67 added/changed/removed files
master: Exporting simple delta revision 6/38 with 0/2/4 added/changed/removed files
master: Exporting simple delta revision 7/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 8/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 9/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 10/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 11/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 12/38 with 0/1/0 added/changed/removed files
master: Exporting thorough delta revision 13/38 with 0/2/0 added/changed/removed files
master: Exporting simple delta revision 14/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 15/38 with 0/2/0 added/changed/removed files
master: Exporting thorough delta revision 16/38 with 0/5/0 added/changed/removed files
master: Exporting simple delta revision 17/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 18/38 with 0/35/5 added/changed/removed files
master: Exporting thorough delta revision 19/38 with 0/36/5 added/changed/removed files
master: Exporting simple delta revision 20/38 with 2/0/0 added/changed/removed files
master: Exporting simple delta revision 21/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 22/38 with 0/1/0 added/changed/removed files
master: Exporting thorough delta revision 23/38 with 0/2/0 added/changed/removed files
master: Exporting simple delta revision 24/38 with 0/1/0 added/changed/removed files
master: Exporting thorough delta revision 25/38 with 0/2/0 added/changed/removed files
master: Exporting simple delta revision 26/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 27/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 28/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 29/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 30/38 with 1/1/0 added/changed/removed files
master: Exporting simple delta revision 31/38 with 0/5/0 added/changed/removed files
master: Exporting thorough delta revision 32/38 with 3/9/0 added/changed/removed files
master: Exporting simple delta revision 33/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 34/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 35/38 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 36/38 with 0/2/0 added/changed/removed files
master: Exporting simple delta revision 37/38 with 0/2/0 added/changed/removed files
master: Exporting simple delta revision 38/38 with 0/1/0 added/changed/removed files
Issued 38 commands
git-fast-import statistics:
---------------------------------------------------------------------
Alloc'd objects:      10000
Total objects:         8561 (       838 duplicates                  )
      blobs  :         7596 (       797 duplicates       4821 deltas of       7333 attempts)
      trees  :          927 (        41 duplicates        219 deltas of        924 attempts)
      commits:           38 (         0 duplicates          0 deltas of          0 attempts)
      tags   :            0 (         0 duplicates          0 deltas of          0 attempts)
Total branches:           1 (         1 loads     )
      marks:           1024 (        38 unique    )
      atoms:           7162
Memory total:          3079 KiB
       pools:          2610 KiB
     objects:           468 KiB
---------------------------------------------------------------------
pack_report: getpagesize()            =       4096
pack_report: core.packedGitWindowSize = 1073741824
pack_report: core.packedGitLimit      = 8589934592
pack_report: pack_used_ctr            =        648
pack_report: pack_mmap_calls          =        167
pack_report: pack_open_windows        =          1 /          1
pack_report: pack_mapped              =   40116445 /   40116445
---------------------------------------------------------------------
{% endhighlight %}

That's the conversion done! Now we just need to clean up a bit.

{% highlight bash %}
$ git clean -f

Removing Makefile
Removing README
Removing hg-fast-export.py
Removing hg-fast-export.sh
Removing hg-reset.py
Removing hg-reset.sh
Removing hg2git.py
Removing hg2git.pyc
Removing svn-archive.c
Removing svn-fast-export.c
Removing svn-fast-export.py
{% endhighlight %}

We've now finished the conversion and cleanup.

### Next Steps

Don't be concerned if you cannot see any files in your newly-converted repository when you try to view them.

{% highlight bash %}
$ ls -a
. .. .git
{% endhighlight %}

This is completely normal. You can verify your repository exists by inspecting your Git objects:

{% highlight bash %}
$ git count-objects -v

count: 0
size: 0
in-pack: 8561
packs: 1
size-pack: 39411
prune-packable: 0
garbage: 0
{% endhighlight %}

You can even do a local filesystem clone of your repository to see what it should look like.

{% highlight bash %}
$ cd ..
$ git clone file://converted-project test-clone

Cloning into 'test-clone'...
remote: Counting objects: 8561, done.
remote: Compressing objects: 100% (3345/3345), done.
remote: Total 8561 (delta 5040), reused 8561 (delta 5040)
Receiving objects: 100% (8561/8561), 38.26 MiB | 34.60 MiB/s, done.
Resolving deltas: 100% (5040/5040), done.

$ cd test-clone
$ ls
Aspnet_Permissions.sql   Itv.Cms.DAL             Itv.Cms.Workflows   Itv.Web.Services.Behaviour
Basic.build              Itv.Cms.Dal.UnitTests   Itv.DMZServices     Itv.Web.UnitTests
...
{% endhighlight %}

### Pushing to GitHub

First, create an empty repository in GitHub.

Now, go to your converted repository and add a new remote that points to the empty GitHub repo.

{% highlight bash %}
$ git remote add origin git@github.com:organisation/repository.git
{% endhighlight %}

We just need to push the converted repository to GitHub now. From the conversion there should only be a master branch (you can check this by running <code>git branch -a</code>).

{% highlight bash %}
$ git push origin master

Counting objects: 8561, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3345/3345), done.
Writing objects: 100% (8561/8561), 38.26 MiB | 677 KiB/s, done.
Total 8561 (delta 5040), reused 8561 (delta 5040)
To git@github.com:organisation/project.git
 * [new branch]      master -> master
{% endhighlight %}

Finished!
