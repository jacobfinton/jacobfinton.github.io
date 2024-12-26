---
layout: post
title:  A Better Way to Manage Multiple Git Profiles
---

## Intro

When setting up a new virtual machine the process can feel tedious without proper automation. Most people familiar with this process will agree that installing base packages and configuring a development environment are necessary, but often time-consuming. During my latest VM build, I realized one thing I had overlooked in my setup was the Git configurations for the projects I had migrated.

While setting the user.name and user.email in Git may seem trivial, this becomes more of a hassle when you're managing multiple platforms—whether for work, school, or personal projects. Each platform requires distinct account information and settings, making it impractical to manually set local configurations for every repository.

Instead of setting local configs for each repo (time-consuming), I was looking for a well-organized and maintainable solution. I decided on a directory-based approach where I group projects by platform. This will allow me to define platform-specific configurations once and have them automatically applied to all relevant repositories in each directory. 

### Repo Structure
```
repos
 |-gitlab
 |-github
 |-work_github
 |-school_github
```

This setup allows me to apply a config for all the projects within a specific directory. I can then personalize each directory with unique profile information and any other platform-specific settings, ensuring that each project has its own tailored configuration while maintaining a simple and consistent structure.

### Config Directory Structure
```bash
git
 |-config # main config that contains sub configs
 |-gitconfig_base
 |-gitconfig_default
 |-gitconfig_gitlab
 |-gitconfig_github 
 |-gitconfig_work_github
 |-gitconfig_school_github
```
The config file would look like this at a minimum. 

```ini
# Base config with shared configurations
[include]
    path = gitconfig_base

# Default profile for anything not covered in sub-configs
[includeif "gitdir:~/repos/"]
    path = gitconfig_default

[includeIf "gitdir:~/repos/gitlab/"]
    path = gitconfig_gitlab

[includeIf "gitdir:~/repos/github/"]
    path = gitconfig_github

[includeIf "gitdir:~/repos/work_github/"]
    path = gitconfig_work_github

[includeIf "gitdir:~/repos/school_github/"]
    path = gitconfig_school_github
```

This approach provides a clean and efficient way to manage multiple profiles based on different directories for various platforms. The base configuration is used for common settings, such as aliases for frequently used Git commands, ensuring consistency across all repositories. A default configuration is included to define a global username and email, which acts as a fallback for any projects that fall outside the current directory structure.

The directory-specific configurations are tailored for individual platforms like GitHub, GitLab, or custom environments. This setup is flexible, allowing you to add new profiles easily as your workflow expands to additional platforms or teams. By organizing the configurations into modular files and leveraging Git’s includeIf functionality, you ensure that your setup is scalable and easy to maintain. This structure also prevents errors by automatically applying the correct settings based on the directory you’re working in.

So far this has met all of my requirements, but I believe there is still room for improvement. While I didn't want to share my exact configuration setup, I’ve provided a template version below that mirrors the example above wiht some random configurations.


```sh
# Paths
MAIN_CONFIG="$HOME/config"
SUBCONFIG_DIR="$HOME/config/git"

# Create the .gitconfigs directory if it doesn't exist
mkdir -p "$SUBCONFIG_DIR"

# Create the main config
cat > "$MAIN_CONFIG" <<EOF
[include]
    path = $SUBCONFIG_DIR/gitconfig_base

# Default profile for anything not covered in subconfigs
[includeIf "gitdir:~/repos/"]
    path = $SUBCONFIG_DIR/gitconfig_default

[includeIf "gitdir:~/repos/gitlab/"]
    path = $SUBCONFIG_DIR/gitconfig_gitlab

[includeIf "gitdir:~/repos/github/"]
    path = $SUBCONFIG_DIR/gitconfig_github

[includeIf "gitdir:~/repos/work_github/"]
    path = $SUBCONFIG_DIR/gitconfig_work_github

[includeIf "gitdir:~/repos/school_github/"]
    path = $SUBCONFIG_DIR/gitconfig_school_github
EOF

# Create the subconfig files for each profile
# Base configuration (common settings)
cat > "$SUBCONFIG_DIR/gitconfig_base" <<EOF
[alias]
    st = status
    co = checkout
    br = branch
    lg = log --oneline --graph --decorate

[core]
    editor = vim
EOF

# Default configuration (global username/email)
cat > "$SUBCONFIG_DIR/gitconfig_default" <<EOF
[user]
    name = Default User
    email = default.user@example.com
EOF

# GitLab profile
cat > "$SUBCONFIG_DIR/gitconfig_gitlab" <<EOF
[user]
    name = GitLab User
    email = gitlab.user@example.com

[core]
    editor = emacs
EOF

# GitHub profile (general)
cat > "$SUBCONFIG_DIR/gitconfig_github" <<EOF
[user]
    name = GitHub User
    email = github.user@example.com
EOF

# Work GitHub profile
cat > "$SUBCONFIG_DIR/gitconfig_work_github" <<EOF
[user]
    name = Work GitHub User
    email = work.github.user@example.com

[credential]
        helper = store
EOF

# School GitHub profile
cat > "$SUBCONFIG_DIR/gitconfig_school_github" <<EOF
[user]
    name = School GitHub User
    email = school.github.user@example.com

[http]
	sslVerify = False
EOF

echo "Git configuration has been successfully set up!"
```

### Resources
1. https://git-scm.com/docs/git-config#_configuration_file
