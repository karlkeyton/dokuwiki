
[Source](https://codingkilledthecat.wordpress.com/2012/04/28/why-your-company-shouldnt-use-git-submodules/ "Permalink to Why your company shouldn’t use Git submodules")

# Why your company shouldn’t use Git submodules

_A programmer had a version control problem and said, "I know, I'll use submodules." Now they have two problems._

It is not uncommon at all when working on any kind of larger-scale project with Git to find yourself wanting to share code between multiple different repositories – whether it be some core system among multiple different products built on top of that system, or perhaps a shared utility library between projects.

At first glance, Git submodules seem to be the perfect answer for this: they come built-in with Git, they act like&nbsp;miniature&nbsp;repositories (so people are already familiar with how to change them), et cetera. They even support pointing at specific versions of the shared code, so if one project doesn't want to deal with integrating the "latest and greatest" version, it doesn't have to.

It's after you've actually worked with submodules for a while that you start to notice just how half-baked Git's submodules system really is.

### Don't blink…

Submodules are&nbsp;effectively separate repositories within the directory tree of their parent repository. The only linkage between the parent and the submodule is recorded value of the submodule's checked-out SHA which is stored in the parent's commits, and changes in that recorded SHA are not automatically reflected in the submodules.

This means that if someone else updates the recorded version of a submodule and you pull their latest changes in the parent repository, your submodule repository will still be pointing to the old version of the submodule. (To update it, you'd need to run `git submodule update`.)

Of course, if you forget to update your submodule to the new version, it's then quite easy to commit the old submodule version in your next parent repository commit – thus effectively reverting the submodule bump by the other developer. Given that submodule changes only show up as 2 commit lines in a diff, it's not hard for such a change to slip by (especially if you're a developer that tends to use `git add .` or `git commit -a` most of the time).

Many code review tools (such as [Review Board][1]) don't support showing submodule changes in code reviews, so an accidental submodule revert isn't likely to get noticed in review, either.

### Merging? Ha!

When Git drops into conflict resolution mode, it still doesn't update the submodule pointers – which means that when you commit the merge after resolving conflicts, you run into the same problem as in the previous section: if you forgot to run `git submodule update`, you've just reverted any submodule commits the branch you merged in might have made.

Furthermore, Git doesn't really handle submodule merging at all. It detects when two changes to the submodule's SHA conflict… but that's it. Since there's no way to have two versions of a submodule checked out at once, it simply doesn't try, effectively treating the entire submodule like a single binary file. It's left to the developer to try to sort out what should be done to get a working submodule out of whatever the branch they're merging in wanted and what their own changes required.

(If you've ever tried to have two people working on a binary file that's tracked in Git, you'll have an idea of how much of a pain it is to resolve such conflicts.)

You typically wind up settling for one of two equally distasteful options: either you have individual branches for submodule changes that mirror the parent repository's branches (so that you can merge the submodule branches when merging the parent's branches), or you force everyone into an effectively Subversion-style linear history of submodule updates with everyone being required to merge in previously added submodule changes before they can make their own.

_There's a reason why I know a lot of people who have nicknamed these things "sobmodules" in their frustration._

### Oh, were you using that?

When you invoke `git submodule update` it looks in the parent repository for a SHA for each submodule, goes into those submodules, and checks out the corresponding SHAs. As would be the case if you checked out a SHA in a regular repository, this puts the submodule into a detached `HEAD` state.

If you then make changes in the submodule and commit then, Git will happily create the commit… and leave you still with a detached `HEAD`. See where this is going yet?

Say you merge in some more changes which happen to include another submodule update. If you haven't committed your own submodule change into the parent project yet, Git won't consider your new commit in the submodule as a conflict, and if you run `git submodule update` it will happily wipe out your commit without warning, replacing it with that from the branch you just merged in.

I hope you had your submodule's reflog enabled or still have the old commit in your terminal scrollback, because otherwise, you just lost all that work you did.

### What am I supposed to do with this?

Submodules acting as almost completely independent repositories has another catch, too – you have to push changes from both the submodule and the parent repository to share with others.

Push changes from the submodule and not the parent repository? No one knows to use your new submodule changes.

Push changes from the parent repository and not the submodule? Congratulations, no one can use your new commits because they don't have the right submodule commit available to check out.

### Well, what else could we do?

So if submodules are such a pain, what are the alternatives? Here's an overview of some of the most popular. Which one is best for you depends on your priorities.

#### Repo

[Repo][2] is a [tool][3] created by Google to manage the rather large Android project, which is spread across multiple different Git project repositories. It essentially works by providing a way to check out multiple projects (Git repositories) in parallel based on a manifest file (which basically serves the purpose that a parent repository does for Git submodules – tracking which submodule commits go together). It also provides a way to submit an atomic changeset that includes changes to multiple different projects.

The downside is that Repo doesn't handle merging very well: it essentially expects you to rebase your changes when you want to bring in outside updates, effectively bringing things back to the equivalent of `svn update`. If you're a fan of many small commits over a few large ones, this can get onerous.

#### Gitslave

[Gitslave][4] is a wrapper around Git that multiplexes git commits into multiple repositories. It effectively implements the "have parallel branches for each of your projects" solution to the merging problem by doing that for you – if you create a branch, it gets created everywhere. If you commit, all of your repositories create a commit, and so on.

Of course, this can get rather hectic if you have a large number of projects and start running into things like merge conflicts in 5 different repositories. It also means you potentially wind up making a lot of pointless extra branches in projects that you didn't happen to touch while touching another project.

#### Git Subtree

[Git Subtree][5] is a tool that uses Git's "subtree merge" functionality to get a similar result to submodules, but via actually storing the files in the main repository and merging in changes directly to that repository.

The upside is that you avoid all the issues with submodule merging because the contents of your subprojects are stored directly in the parent repository and thus are treated like any other tracked files when pulling and&nbsp;merging.

The downside is that all of your subproject files are present in the parent repository, which means you're giving up some of the reason for originally splitting up your project repositories: having one canonical repository for a given set of shared code. If someone makes a change to a subproject, they can merge it with other changes locally, but they'd have to explicitly split that change back out of their project if they wanted to share it with projects.

#### Others

A couple of other potential options are [Braid][6] and [giternal][7], both of which offer a more svn-externals kind of external dependency linking (in the sense that you can ask it to grab the latest version of a given repository's contents and place it in your tree).

[1]: http://www.reviewboard.org
[2]: http://source.android.com/source/version-control.html
[3]: http://source.android.com/source/using-repo.html
[4]: http://gitslave.sourceforge.net/
[5]: https://github.com/apenwarr/git-subtree/
[6]: https://github.com/evilchelu/braid/wiki
[7]: https://github.com/patmaddox/giternal
  
