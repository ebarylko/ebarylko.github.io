# my-blog

## Building the Blog
*Run `lein run`

## Serving the Site Locally
* Run `clojure -X:serve`

## Adding New Posts
* Create an issue in github describing what the post will be about
* Create a new local branch with the following format: `post-xx-your-title-goes-here` where `xx` is the issue number
* Add the post in the `content/md/posts` folder
* Verify how the post looks locally
* Commit and push
* Create a pull request by running `gh pr create` with the following sections: 
```
## Summary
Brief description of the post

## Changes
// Follows the format of having first the file changed, and changes made below

### First File Change
- Change 1
- Change 2

### Second File Change
- Change 1
- Change 2


```

* Review and merge the pull request

* Go to main branch and run `gh pr create --base live`
* When approving merge, in the comments say that it closes #"whatever was the number of the issue you opened"
