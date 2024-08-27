---
title: Separate Folder from Git Repository with Complete History
description: You can turn a folder within a Git repository into a brand new repository. 
categories: [demo, tech]
date: 2024-08-27 09:18:00 +500
tags: [git]     # TAG names should always be lowercase
pin: false
math: true
mermaid: true
image:
  path: https://ik.imagekit.io/yhmzg9hf6/git-blog-header.png?updatedAt=1724738368191 
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: "Git"
---

## Splitting a subfolder out into a new git repository

> Before proceeding with splitting a subfolder into a new git repository, make sure to create a backup of your existing repository. This will help you preserve the complete history and avoid any potential data loss. You need Git version 2.22.0 or later to follow these instructions, otherwise `git filter-repo` will not work.
{: .prompt-warning }

If you create a new clone of the repository, you won't lose any of your Git history or changes when you split a folder into a separate repository. However, note that the new repository won't have the branches and tags of the original repository.

1. Open a Git Bash, Terminal or Command Prompt.
2. Change the current working directory to the location where you want to create your new repository.
3. Clone the repository that contains the subfolder.

   ```bash
   git clone https://github.com/USERNAME/REPOSITORY-NAME
   ```

4. Change the current working directory to your cloned repository.
   ```bash
   cd REPOSITORY-NAME
   ```

> Windows users should use `/` to delimit folders. 
{: .prompt-tip }

5. To filter out the subfolder from the rest of the files in the repository, install [git-filter-repo](https://github.com/newren/git-filter-repo), then run `git filter-repo` with the following arguments.
    * `FOLDER-NAME`: The folder within your project where you'd like to create a separate repository.
    
    ```bash
    git filter-repo --path FOLDER1-NAME/ --path FOLDER2-NAME/
    # Filter the specified branch in your directory and remove empty commits
    ```
    The repository should now only contain the files that were in your subfolder(s).
    If you want one specific subfolder to be the new root folder of the new repository, you can use the following command:
    ```bash
    git filter-repo --subdirectory-filter FOLDER-NAME
    # Filter the specific branch by using a single sub-directory as the root for the new repository
    ```

6. Create a new repository on GitHub/GitLab.
7. Add a new remote name with the URL you copied for your repository. For example, origin or upstream are two common choices.
    ```bash
    git remote add origin https://github.com/USERNAME/REPOSITORY-NAME.git
    ```
8. Verify that the remote URL was added with your new repository name.
    ```bash
    git remote -v
    # Verify new remote URL
    origin  https://github.com/USERNAME/NEW-REPOSITORY-NAME.git (fetch)
    origin  https://github.com/USERNAME/NEW-REPOSITORY-NAME.git (push)
    ```

9. Push your changes to the new repository on GitHub.
    ```bash
    git push -u origin BRANCH-NAME
    ```