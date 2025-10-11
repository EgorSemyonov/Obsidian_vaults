There is a pointer used in Git. This is a reference to a particular commit in the current branch. Every branch has its own pointer.

Usually the pointer refers to the last commit. We can move the pointer to another commit by the command:
```bash
git reset
```
There are 3 options how we can do it:
1) `--soft` - do not remove local changes and keeps them staged in the [[Index]]
2) `--hard` - deletes all local changes that have been made after new pointer
3) `--mixed` - do not remove local changes but delete them from [[Index]]

There is a key word `HEAD`. It's a pointer to current commit. We can veiw this pointer by the command:
```bash
git show HEAD
```

We can also change the pointer by the modernazing of key word `HEAD`:
- `HEAD` - where I am now
- `HEAD~` - one commit before where I am now
- `HEAD~2` - two commits before where I am now

The whole command to delete last commit:
```bash
git reset --hard HEAD~
git push --force
```