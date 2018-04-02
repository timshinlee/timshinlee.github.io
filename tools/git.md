### tag

// setting user
git config --global user.name "timshinlee"
git config --global user.email "timshinlee@gmail.com"

// init current directory as a git directory
git init

// add file
git add .

// commit changes
git commit -m "initial commit"

creating tags: `git tag tag_name`
```
git tag 1.0.0
```

pushing tags: `git push remote_repository_name tag_name`
```
git push origin 1.0.0
```
