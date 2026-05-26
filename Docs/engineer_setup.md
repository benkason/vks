# Engineer Setup (Work-in-progress)

These are steps on how a new member of the platform team can set up their
environment.

- Install VSCode https://code.visualstudio.com/download
- Puttygen - make a key, save public+private key.  Private key needs to be PEM format (in PuttyGen, Conversions > Export OpenSSH key)
- Install Git bash
- VSCode extensions: Remote-SSH, shellcheck
- Git clone the repo git@github.com:MTNNigeria/ng-tkgi-platform.git

# Using VSCode
- Open VSCode
- Navigate to the folder where you have cloned the repository
- Open Terminal and switch it to Git bash
- You can set default profile as git bash by using "Select Default Profile" under the same arrow and select Git bash.
![Gitbash_terminal](./images/Gitbash_terminal.png)

# To check Current Repo
- See git config using the following Command
```bash
cat .git/config
```
The url below show your current repo you are working in.
```
[remote "origin"]
        
        url = git@github.com:MTNNigeria/ng-tkgi-platform.git

```

# How to the save code/changes and commit to git hub repo.

- git status
- git add docs/images/Gitbash_terminal.png
- git add docs/engineer_setup.md
- git commit 
- git pull
- git push

# Trouble shooting steps to resolve access rights
- If you get the permissions error as descirbed in screen shot below
![permissions_error](./images/permissions_error.png)

- Generate a new SSH key
ssh-keygen -t rsa
![sshkey](./images/sample_sshkey.png)
and then  add it to your profile

- The issue that may occur is the SSH agent is not running
so you need to do the following procedure.
![ssh_agent_tshoot](./images/ssh_agent_tshoot.png)
![ssh-add](./images/ssh-add.png)
![git_pull](./images/git_pull.png)