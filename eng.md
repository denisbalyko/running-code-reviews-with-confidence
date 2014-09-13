Growing up, I learned there were two kinds of reviews I could seek out from my
parents. One parent gave reviews in the form of a shower of praise. The other 
parent, the one with a degree from the Royal College of Art, would put me 
through a design crit. Today the reviews I seek are for my code, not my horse 
drawings, but it continues to be a process I both dread and crave.<aside class
="content-minutiae
">

## Illustration:

## Share This:</aside>

In this article, I’ll describe my battle-tested process for conducting code
reviews, highlighting the questions you should ask during the review process as 
well as the necessary version control commands to download and review someone’s 
work. I’ll assume your team uses Git to store its code, but the process works 
much the same if you’re using any other source control system.

Completing a peer review is time-consuming. In the last project where I
introduced mandatory peer reviews, the senior developer and I estimated that it 
doubled the time to complete each ticket. The reviews introduced more context-
switching for the developers, and were a source of increased frustration when it
came to keeping the branches up to date while waiting for a code review.

The benefits, however, were huge. Coders gained a greater understanding of the
whole project through their reviews, reducing silos and making onboarding easier
for new people. Senior developers had better opportunities to ask why decisions 
were being made in the codebase that could potentially affect future work. And 
by adopting an ongoing peer review process, we reduced the amount of time needed
for human quality assurance testing at the end of each sprint.

Let’s walk through the process. Our first step is to figure out exactly what
we’re looking for.

## Determine the purpose of the proposed change

Our code review should always begin in a ticketing system, such as Jira or
GitHub. It doesn’t matter if the proposed change is a new feature, a bug fix, a 
security fix, or a typo: every change should start with a description of why the
change is necessary, and what the desired outcome will be once the change has 
been applied. This allows us to accurately assess when the proposed change is 
complete.

The ticketing system is where you’ll track the discussion about the changes
that need to be made after reviewing the proposed work. From the ticketing 
system, you’ll determine which branch contains the proposed code. Let’s pretend 
the ticket we’re reviewing today is 61524—it was created to fix a broken link in
our website. It could just as equally be a refactoring, or a new feature, but I’
ve chosen a bug fix for the example. No matter what the nature of the proposed 
change is, having each ticket correspond to only one branch in the repository 
will make it easier to review, and close, tickets.

Set up your local environment and ensure that you can reproduce what is
currently the live site—complete with the broken link that needs fixing. When 
you apply the new code locally, you want to catch any regressions or problems it
might introduce. You can only do this if you know, for sure, the difference 
between what is old and what is new.

## Review the proposed changes

At this point you’re ready to dive into the code. I’m going to assume you
’re working with Git repositories, on a branch-per-issue setup, and that the 
proposed change is part of a remote team repository. Working directly from the 
command line is a good universal approach, and allows me to create copy-paste 
instructions for teams regardless of platform.

To begin, update your local list of branches.

    git fetch
    

Then list all available branches.

    git branch -a
    

A list of branches will be displayed to your terminal window. It may appear
something like this:

    * master
    remotes/origin/master
    remotes/origin/HEAD -> origin/master
    remotes/origin/61524-broken-link
    

The `*` denotes the name of the branch you are currently viewing (or have “
checked out”). Lines beginning with`remotes/origin` are references to branches
we’ve downloaded. We are going to work with a new, local copy of branch
`61524-broken-link`.

When you clone your project, you’ll have a connection to the remote
repository as a whole, but you won’t have a read-write relationship with each of
the individual branches in the remote repository. You’ll make an explicit 
connection as you switch to the branch. This means if you need to run the 
command`git push` to upload your changes, Git will know which remote repository
you want to publish your changes to.

    git checkout --track origin/61524-broken-link
    

Ta-da! You now have your own copy of the branch for ticket 61524, which is
connected (“tracked”) to the origin copy in the remote repository. You can now 
begin your review!

First, let’s take a look at the commit history for this branch with the
command`log`.

    git log master
    

Sample output:

    Author: emmajane <emma>
    Date: Mon Jun 30 17:23:09 2014 -0400
    
    Link to resources page was incorrectly spelled. Fixed.
    
    Resolves #61524.</emma>
    

This gives you the full log message of all the commits that are in the branch
`61524-broken-link`, but are not also in the `master` branch. Skim through the
messages to get a sense of what’s happening.

Next, take a brief gander through the commit itself using the `diff` command.
This command shows the difference between two snapshots in your repository. You 
want to compare the code on your checked-out branch to the branch you’ll be 
merging “to”—which conventionally is the`master` branch.

    git diff master
    

### How to read patch files

When you run the command to output the difference, the information will be
presented as a patch file. Patch files are ugly to read. You’re looking for 
lines beginning with`+` or `-`. These are lines that have been added or removed
, respectively. Scroll through the changes using the up and down arrows, and 
press`q` to quit when you’ve finished reviewing. If you need an even more
concise comparison of what’s happened in the patch, consider modifying the diff 
command to list the changed files, and then look at the changed files one at a 
time:

    git diff master --name-only
    git diff master <filename>
    

Let’s take a look at the format of a patch file.

    diff --git a/about.html b/about.html
    index a3aa100..a660181 100644
    	--- a/about.html
    	+++ b/about.html
    @@ -48,5 +48,5 @@
    	(2004-05)
    
    - A full list of <a href="emmajane.net/events">public 
    + A full list of <a href="http://emmajane.net/events">public 
    presentations and workshops</a> Emma has given is available
    

I tend to skim past the metadata when reading patches and just focus on the
lines that start with`-` or `+`. This means I start reading at the line
immediate following`@@`. There are a few lines of context provided leading up
to the changes. These lines are indented by one space each. The changed lines of
code are then displayed with a preceding`-` (line removed), or `+` (line added
).

### Going beyond the command line

Using a Git repository browser, such as gitk, allows you to get a slightly
better visual summary of the information we’ve looked at to date. The version of
Git that Apple ships with does not include gitk—I used[Homebrew][1] to re-
install Git and get this utility. Any repository browser will suffice, though, 
and there are many[GUI clients available on the Git website][2].

    gitk
    

When you run the command `gitk`, a graphical tool will launch from the command
line. An example of the output is given in the following screenshot. Click on 
each of the commits to get more information about it. Many ticket systems will 
also allow you to look at the changes in a merge proposal side-by-side, so if 
you’re finding this cumbersome, click around in your ticketing system to find 
the comparison tools they might have—I know for sure GitHub offers this feature.

![Screenshot of the gitk repository browser.][3]</figure>
Now that you’ve had a good look at the code, jot down your answers to the
following questions:

1.  Does the code comply with your project’s identified coding standards?
2.  Does the code limit itself to the scope identified in the ticket?
3.  Does the code follow industry best practices in the most efficient way
    possible?
   
4.  Has the code been implemented in the best possible way according to all of
    your internal specifications? It’s important to separate your preferences and 
    stylistic differences from actual problems with the code.
   

## Apply the proposed changes

Now is the time to start up your testing environment and view the proposed
change in context. How does it look? Does your solution match what the coder 
thinks they’ve built? If it doesn’t look right, do you need to clear the cache, 
or perhaps rebuild the Sass output to update the CSS for the project?

Now is the time to also test the code against whatever test suite you use.

1.  Does the code introduce any regressions?
2.  Does the new code perform as well as the old code? Does it still fall
    within your project’s performance budget for download and page rendering times?
   
3.  Are the words all spelled correctly, and do they follow any brand-specific
    guidelines you have?
   

Depending on the context for this particular code change, there may be other
obvious questions you need to address as part of your code review.

Do your best to create the most comprehensive list of everything you can find
wrong (and right) with the code. It’s annoying to get dribbles of feedback from 
someone as part of the review process, so we’ll try to avoid “just one more 
thing” wherever we can.

## Prepare your feedback

Let’s assume you’ve now got a big juicy list of feedback. Maybe you have no
feedback, but I doubt it. If you’ve made it this far in the article, it’s 
because you love to comb through code as much as I do. Let your freak flag fly 
and let’s get your review structured in a usable manner for your teammates.

For all the notes you’ve assembled to date, sort them into the following
categories:

1.  The code is broken. It doesn’t compile, introduces a regression, it doesn
    ’t pass the testing suite, or in some way actually fails demonstrably. These are
    problems which absolutely must be fixed.
   
2.  The code does not follow best practices. You have some conventions, the web
    industry has some guidelines. These fixes are pretty important to make, but they
    may have some nuances which the developer might not be aware of.
   
3.  The code isn’t how you would have written it. You’re a developer with
    battle-tested opinions, and you know you’re right, you just haven’t had the 
    chance to update the Wikipedia page yet to prove it.
   

## Submit your evaluation

Based on this new categorization, you are ready to engage in passive-aggressive
coding. If the problem is clearly a typo and falls into one of the first two 
categories, go ahead and fix it. Obvious typos don’t really need to go back to 
the original author, do they? Sure, your teammate will be a little embarrassed, 
but they’ll appreciate you having saved them a bit of time, and you’ll increase 
the efficiency of the team by reducing the number of round trips the code needs 
to take between the developer and the reviewer.

If the change you are itching to make falls into the third category: stop. Do
not touch the code. Instead, go back to your colleague and get them to describe 
their approach. Asking “why” might lead to a really interesting conversation 
about the merits of the approach taken. It may also reveal limitations of the 
approach to the original developer. By starting the conversation, you open 
yourself to the possibility that just maybe your way of doing things isn’t the 
only viable solution.

If you needed to make any changes to the code, they should be absolutely tiny
and minor. You should not be making substantive edits in a peer review process. 
Make the tiny edits, and then add the changes to your local repository as 
follows:

    git add .
    git commit -m "[#61524] Correcting <list problem> identified in peer review."
    

You can keep the message brief, as your changes should be minor. At this point
you should push the reviewed code back up to the server for the original 
developer to double-check and review. Assuming you’ve set up the branch as a 
tracking branch, it should just be a matter of running the command as follows:

    git push
    

Update the issue in your ticketing system as is appropriate for your review.
Perhaps the code needs more work, or perhaps it was good as written and it is 
now time to close the issue queue.

Repeat the steps in this section until the proposed change is complete, and
ready to be merged into the main branch.

## Merge the approved change into the trunk

Up to this point you’ve been comparing a ticket branch to the master branch
in the repository. This main branch is referred to as the “trunk” of your 
project. (It’s a tree thing, not an elephant thing.) The final step in the 
review process will be to merge the ticket branch into the trunk, and clean up 
the corresponding ticket branches.

Begin by updating your master branch to ensure you can publish your changes
after the merge.

    git checkout master
    git pull origin master
    

Take a deep breath, and merge your ticket branch back into the main repository
. As written, the following command will not create a new commit in your 
repository history. The commits will simply shuffle into line on the master 
branch, making`git log −−graph` appear as though a separate branch has never
existed. If you would like to maintain the illusion of a past branch, simply add
the parameter`−−no-ff` to the merge command, which will make it clear, via the
graph history and a new commit message, that you have merged a branch at this 
point. Check with your team to see what’s preferred.

    git merge 61524-broken-link
    

The merge will either fail, or it will succeed. If there are no merge errors,
you are ready to share the revised master branch by uploading it to the central 
repository.

    git push
    

If there are merge errors, the original coders are often better equipped to
figure out how to fix them, so you may need to ask them to resolve the conflicts
for you.

Once the new commits have been successfully integrated into the master branch,
you can delete the old copies of the ticket branches both from your local 
repository and on the central repository. It’s just basic housekeeping at this 
point.

    git branch -d 61524-broken-link
    git push origin --delete 61524-broken-link
    

## Conclusion<aside class="sidebar">

## Also in Issue № 402

## [Git: The Safety Net for Your Projects][4]

by Tobias Günther</aside>

This is the process that has worked for the teams I’ve been a part of.
Without a peer review process, it can be difficult to address problems in a 
codebase without blame. With it, the code becomes much more collaborative; when 
a mistake gets in, it’s because we both missed it. And when a mistake is found 
before it’s committed, we both breathe a sigh of relief that it was found when 
it was.

Regardless of whether you’re using Git or another source control system, the
peer review process can help your team. Peer-reviewed code might take more time 
to develop, but it contains fewer mistakes, and has a strong, more diverse team 
supporting it. And, yes, I’ve been known to learn the habits of my reviewers and
choose the most appropriate review style for my work, just like I did as a kid.

 [1]: http://brew.sh/
 [2]: http://git-scm.com/downloads/guis
 [3]: img/gitk.jpg
 [4]: http://alistapart.com/article/git-the-safety-net-for-your-projects