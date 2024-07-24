# Should you be deleting your merges branches?

Git is a version control system that allows you to keep track of your projects history. One of the most notable features of git is branching. Branching allows you to split your git history into multiple 'branches', each maintaining a separate version of your project. Usually branches eventually get merged back into mainstream. Most, if not all major git providers, such as GitHub, GitLab and BitBucket, provide an option to delete a branch after it has been merged, usually in form of a checkbox near the 'merge PR' button. If this checkbox is ticked, then your branch gets merged and then deleted. But what are the implications of doing this? Should you be deleting your git branches?

The short answer is that probably yes. There's usually no reason to not delete a branch

Here are some reasons why you should be deleting your branches:

**Merged branches pollute the branch list**

If you kept all your branches around, it would be difficult to navigate the branch list and find the branches that are currently being worked on. Deleting old merged branches will make your branch list clean and easy to navigate

**Deleting a merged branch doensn't delete the commits**

A branch is nothing but a reference or pointer to the latest commit in that branch. When you delete a merged branch, only the pointer is deleted, but the history and the commit itself is kept and can be accessed just like any other commit. **Keep in mind that deleting a non-merged branch *will* delete all of the associated commits on that branch**