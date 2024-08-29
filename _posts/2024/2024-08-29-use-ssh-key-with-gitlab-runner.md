---
title: Using SSH Private Keys with GitLab Runner
description: You can turn a private key in docker runner to clone reposotries and ssh other machines
categories: [demo, tech]
date: 2024-08-29 09:45:00 +500
tags: [gitlab, git]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
image:
  path: https://ik.imagekit.io/yhmzg9hf6/gitlab-runner.png?updatedAt=1724906195090
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: "GitLab"
---
## Using SSH keys with GitLab Runner

Using SSH keys with a GitLab Runner allows the runner to securely authenticate and interact with private repositories. This can be particularly useful for cloning repositories and accessing private submodules in CI/CD pipelines. Here’s how you can set up SSH keys for your GitLab Runner:

## Generate an SSH Key Pair

If you haven’t already generated an SSH key pair, you can do so on the machine where the GitLab Runner is installed:

  1. Open a terminal on the GitLab Runner machine.
  2. Generate an SSH key pair using the following command:
      ```bash
      ssh-keygen -t ed25519 -C "your_email@example.com"

      ```
  3. Press Enter to accept the default file location (~/.ssh/id_ed25519).
  4. Enter a passphrase if you want additional security (optional but recommended).
  5. You should now have two files:
      * `~/.ssh/id_ed25519` (private key)
      * `~/.ssh/id_ed25519.pub` (public key)


## Add the SSH Public Key to GitLab

1. Copy the public key to your clipboard:
    ```bash
    Copy the public key to your clipboard:
    ```

2. Log in to your GitLab account and go to your Profile Settings.
3. Navigate to SSH Keys and paste your SSH public key into the “Key” field.
4. Give it a name and set an expiration date if desired, then click Add key.

## Configure GitLab Runner to Use the SSH Key

To use the SSH key with your GitLab Runner, you need to configure the runner to use this SSH key for authentication.

### Private Runner

1. Open the terminal on the GitLab Runner machine.
2. Add the SSH key to the SSH agent:
    ```bash
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/id_ed25519
    ```

### Configure the SSH key in the `config.toml` file of the GitLab Runner

1. Open the GitLab Runner configuration file (usually located at `/etc/gitlab-runner/config.toml` or `C:\GitLab-Runner\config.toml` on Windows).
2. Find the section for your runner (identified by the `[[runners]]` heading).
3. Add or modify the `pre_clone_script` to load the SSH key before running any Git commands:
    ```bash
    [[runners]]
    name = "my-runner"
    url = "https://gitlab.com/"
    token = "RUNNER_TOKEN"
    executor = "shell"
    [runners.custom_build_dir]
    [runners.cache]
      [runners.cache.s3]
      [runners.cache.gcs]
    pre_clone_script = """
      eval $(ssh-agent -s)
      ssh-add ~/.ssh/id_ed25519
    """
    ```
4. Save and close the `config.toml` file.
5. Restart the runner service or runner.


### Use SSH in Your `.gitlab-ci.yml`

In your `.gitlab-ci.yml`, use the git protocol for cloning repositories, and ensure your job does not prompt for SSH key passphrases.
#### Example code for setting up keys

```yml
image: ubuntu

before_script:
  ##
  ## Install ssh-agent if not already installed, it is required by Docker.
  ## (change apt-get to yum if you use an RPM-based image)
  ##
  - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client git -y )'

  ##
  ## Run ssh-agent (inside the build environment)
  ##
  - eval $(ssh-agent -s)

  ##
  ## Add the SSH key stored in SSH_PRIVATE_KEY variable to the agent store
  ## We're using tr to fix line endings which makes ed25519 keys work
  ## without extra base64 encoding.
  ## https://gitlab.com/gitlab-examples/ssh-private-key/issues/1#note_48526556
  ##
  - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -

  ##
  ## Create the SSH directory and give it the right permissions
  ##
  - mkdir -p ~/.ssh
  - chmod 700 ~/.ssh

  ##
  ## Use ssh-keyscan to scan the keys of your private server. Replace gitlab.com
  ## with your own domain name. You can copy and repeat that command if you have
  ## more than one server to connect to.
  ##
  - ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
  - chmod 644 ~/.ssh/known_hosts

  ##
  ## Alternatively, assuming you created the SSH_SERVER_HOSTKEYS variable
  ## previously, uncomment the following two lines instead.
  ##
  #- echo "$SSH_SERVER_HOSTKEYS" > ~/.ssh/known_hosts
  #- chmod 644 ~/.ssh/known_hosts

  ##
  ## You can optionally disable host key checking. Be aware that by adding that
  ## you are suspectible to man-in-the-middle attacks.
  ## WARNING: Use this only with the Docker executor, if you use it with shell
  ## you will overwrite your user's SSH config.
  ##
  #- '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'

  ##
  ## Optionally, if you will be using any Git commands, set the user name and
  ## email.
  ##
  - git config --global user.email "darth@empire.com"
  - git config --global user.name "Darth Vader"


Test SSH:
  script:
  # try to connect to GitLab.com
  - ssh git@gitlab.com

  # try to clone yourself. A *PUBLIC* key paired to the SSH_PRIVATE_KEY was added as deploy key to this repository
  - git clone git@gitlab.com:gitlab-examples/ssh-private-key.git
```

## Additional Notes
* **Permissions**: Ensure the SSH keys are set with correct permissions. The private key should have `600` permissions, and the `.ssh` directory should have `700` permissions.
* **SSH Key Scanning**: You may want to add GitLab's SSH fingerprint to ~/.ssh/known_hosts to prevent any prompt during the first connection.
* **Private Repositories**: If you are cloning a private repository, the SSH key must be added to the user account that has access to that repository.

By following these steps, you can securely use SSH keys with your GitLab Runner to interact with Git repositories and perform various tasks.