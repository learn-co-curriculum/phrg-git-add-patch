# Git Add --Patch

`git` is a powerful and complex CLI that manages versions of your code. This lesson explores using the `git add --patch` option.

## The Manual

It is helpful to utilize [`git` documentation](https://git-scm.com/doc) often as you grow as a developer. Sometimes it is even easier to launch `git` documentation in your shell session. Let's do this right now for `git add`, using the `--help` option. You can pass the `--help` option to any `git` command for more information.

```
git add --help
```

Take a few minutes to read about some of the ways one can pass additional options and configuration to `git add`.

---

## Git Patch

To stage our code, we have been using `git add <path>`, where `<path>` is the location of the file or directory we wish to stage. Often we wish to stage all of our changes, so we use `git add .`. But sometimes we make changes across an application. To divide our commits up logically we need to stage only select changes. In some cases, select changes contained in a single file. This is where the `--patch` option comes handy.

## Why

Creating logical commits facilitates an easier code review for our team. This means they do a better job understanding your changes, offer better feedback, and make it easy for them to detect a possible bug, which can be a life saver for our application. It also creates a better git history for our project. A few weeks or months from now a developer may need to look back at the git history to identify how a feature was added.

### Example

For our reptile PR, we've added `:show` views for our Lizard and Turtle resources. We completed the lizard show logic first, then the turtle logic, but forgot to commit in between. We go to commit our lizard files first, but notice that our `routes.rb` contains logic for both reptiles. `git diff` returns:

```
$ git diff
diff --git a/config/routes.rb b/config/routes.rb
index f4d4adc..927f7f9 100644
--- a/config/routes.rb
+++ b/config/routes.rb
@@ -4,7 +4,7 @@

 resources :cats
 resources :dogs
-resources :lizards, only: [:index, :create]
+resources :lizards, only: [:index, :create, :show]
 resources :ninjas
-resources :turtles, only: [:index, :create]
+resources :turtles, only: [:index, :create, :show]
 resources :xylophones
```

For our first commit, all we want is the line of code that adds the `:show` route to the `:lizards` resource. We do not want the new `:turtles` route.

`git add --patch` gives us the ability to stage only a "hunk" of the file.

```
$ git add --patch config/routes.rb
diff --git a/config/routes.rb b/config/routes.rb
index f4d4adc..927f7f9 100644
--- a/routes.rb
+++ b/routes.rb
@@ -4,7 +4,7 @@

 resources :cats
 resources :dogs
-resources :lizards, only: [:index, :create]
+resources :lizards, only: [:index, :create, :show]
 resources :ninjas
-resources :turtles, only: [:index, :create]
+resources :turtles, only: [:index, :create, :show]
 resources :xylophones
Stage this hunk [y,n,q,a,d,s,e,?]?
```

Using the `--patch` option launches Git's Interactive Mode, providing the user prompts with what to do next. Here we are prompted with a number of options for how to continue, `[y,n,q,a,d,s,e,?]`. Let's use the `?` option to see what each of these mean. Picking an option is as easy as typing the character and pressing Enter.

```
Stage this hunk [y,n,q,a,d,s,e,?]? ?
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk or any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk or any of the later hunks in the file
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
@@ -4,7 +4,7 @@

 resources :cats
 resources :dogs
-resources :lizards, only: [:index, :create]
+resources :lizards, only: [:index, :create, :show]
 resources :ninjas
-resources :turtles, only: [:index, :create]
+resources :turtles, only: [:index, :create, :show]
 resources :xylophones
Stage this hunk [y,n,q,a,d,s,e,?]?
```

Based on the hunk provided, it looks like if we used `y` we would still be staging too many of our changes. So let's `split` this hunk with the `s` option:

```
Stage this hunk [y,n,q,a,d,s,e,?]? s
Split into 2 hunks.
@@ -4,5 +4,5 @@

 resources :cats
 resources :dogs
-resources :lizards, only: [:index, :create]
+resources :lizards, only: [:index, :create, :show]
 resources :ninjas
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]?
```

`git` has returned that it `Split into 2 hunks.`, and now only our `:lizards` diff is reported. This looks good! Lets stage it:


```
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? y
@@ -8,3 +8,3 @@
 resources :ninjas
-resources :turtles, only: [:index, :create]
+resources :turtles, only: [:index, :create, :show]
 resources :xylophones
```

After specifying `y`, `git` moves on to it's next hunk. But we do not want to stage the `:turtles` line, so we will specify `n`:

```
Stage this hunk [y,n,q,a,d,K,g,/,e,?]? n

$
```

Having gotten to the end of the changes in our `routes.rb`, our prompts stop, and we exit back to the command line. Let's verify that we've only staged the lizard logic with `git status` and `git diff`:

```
$ git status config/routes.rb
On branch reptiles
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

  modified:   config/routes.rb

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

  modified:   config/routes.rb

$ git diff config/routes.rb
diff --git a/config/routes.rb b/config/routes.rb
index 8ebdb30..927f7f9 100644
--- a/config/routes.rb
+++ b/config/routes.rb
@@ -6,5 +6,5 @@ resources :cats
 resources :dogs
 resources :lizards, only: [:index, :create, :show]
 resources :ninjas
-resources :turtles, only: [:index, :create]
+resources :turtles, only: [:index, :create, :show]
 resources :xylophones
```

`git status config/routes.rb` reveals that our `routes.rb` has both staged and unstaged changes (good!). `git diff config/routes.rb` reveals that only the `:turtles` line is unstaged (perfect!). We are good to continue adding files until we are ready to create our `:lizards` focused commit.

## Resources

- [`git` documentation](https://git-scm.com/doc)
- [`git add --patch` documentation](https://git-scm.com/docs/git-add#git-add--p)
