Here’s a *detailed step-by-step guide* to syncing a *private GitHub repository* to a *self-hosted GitLab* *for free* using a *cron job*.

---

# *Overview*
This method involves:
1. Setting up *SSH authentication* between GitHub, GitLab, and your server.
2. *Cloning the GitHub repository* in *mirror mode*.
3. *Configuring GitLab as the push destination*.
4. *Automating the sync* using a *cron job*.

---

# *Step 1: Prepare Your Server (or Local Machine)*
You need a machine to run the sync script. This can be:
- A *Linux server* (e.g., a VPS, dedicated server, or cloud instance).
- Your *own computer* (if it's always online).
- A *self-hosted GitLab Runner*.

> 🛠 *Install Git on Your Server* (if not already installed):
sh
sudo apt update && sudo apt install git -y   # Ubuntu/Debian
sudo yum install git -y                      # CentOS/RHEL


---

# *Step 2: Set Up SSH Authentication*
Since the GitHub repository is *private, we need **SSH authentication* to allow the server to *fetch updates* from GitHub and *push* them to GitLab.

## *2.1 Generate an SSH Key Pair*
Run the following command on your server:
sh
ssh-keygen -t rsa -b 4096 -C "git-sync" -f ~/.ssh/git_sync_key

This will generate:
- A *private key* → ~/.ssh/git_sync_key
- A *public key* → ~/.ssh/git_sync_key.pub

---

## *2.2 Add the Public Key to GitHub*
1. Go to *GitHub* → *Your Repo* → *Settings* → *Deploy Keys*.
2. Click *"Add Deploy Key"*.
3. Paste the *content of ~/.ssh/git_sync_key.pub*.
4. Enable *Read access* (✅ Allow Read Access).

---

## *2.3 Add the Public Key to GitLab*
1. Go to *GitLab* → *Your Repo* → *Settings* → *Deploy Keys*.
2. Click *"Add Deploy Key"*.
3. Paste the *content of ~/.ssh/git_sync_key.pub*.
4. Enable *Write access* (✅ Allow Write Access).

---

## *2.4 Configure SSH to Use This Key*
Edit your *SSH config file*:
sh
nano ~/.ssh/config

Add the following:

Host github
  HostName github.com
  User git
  IdentityFile ~/.ssh/git_sync_key
  StrictHostKeyChecking no

Host gitlab
  HostName gitlab.yourdomain.com
  User git
  IdentityFile ~/.ssh/git_sync_key
  StrictHostKeyChecking no

Save and exit (CTRL + X, then Y, then Enter).

---

## *2.5 Test SSH Connections*
Test GitHub:
sh
ssh -T git@github.com

Expected output:

Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.

Test GitLab:
sh
ssh -T git@gitlab.yourdomain.com

Expected output:

Welcome to GitLab, your-username!


If both tests succeed, SSH authentication is *correctly configured*.

---

# *Step 3: Clone the GitHub Repository*
Clone the repository using *mirror mode* (this copies all branches and tags):
sh
git clone --mirror git@github.com:your-org/repo.git

Move into the cloned directory:
sh
cd repo.git


---

# *Step 4: Add GitLab as a Remote Repository*
Set GitLab as the new *push* destination:
sh
git remote set-url --push origin git@gitlab.yourdomain.com:your-org/repo.git

Verify the remotes:
sh
git remote -v

Expected output:

origin  git@github.com:your-org/repo.git (fetch)
origin  git@gitlab.yourdomain.com:your-org/repo.git (push)

- *Fetch from GitHub*
- *Push to GitLab*

---

# *Step 5: Sync Changes Manually (Test Run)*
To test the sync process, run:
sh
git fetch -p origin   # Pull new changes from GitHub
git push --mirror     # Push all changes to GitLab

If this works without errors, proceed to automation.

---

# *Step 6: Automate Syncing with a Cron Job*
Now, we automate the process to run *every hour*.

## *6.1 Open the Crontab Editor*
sh
crontab -e


## *6.2 Add the Sync Job*
Add this line at the bottom:
sh
0 * * * * cd /path/to/repo.git && git fetch -p origin && git push --mirror

### *Explanation:*
- 0 * * * * → Runs **every hour** at minute 0.
- cd /path/to/repo.git → Moves to the repository directory.
- git fetch -p origin → Fetches changes from GitHub.
- git push --mirror → Pushes changes to GitLab.

## *6.3 Save and Exit*
- If using *nano*, press CTRL + X, then Y, then Enter.

---

# *Step 7: Verify the Cron Job*
To confirm that the cron job is running, check logs:
sh
cat /var/log/syslog | grep CRON

If you see entries related to your sync command, it's working.

---

# *🎯 Final Result*
✅ *GitHub Private Repo is mirrored to GitLab for free.*  
✅ *No need for GitHub Actions or paid GitLab features.*  
✅ *Runs automatically every hour.*  

---

# *📌 Optional: Sync More Often*
- To *sync every 15 minutes*, change:
  sh
  */15 * * * * cd /path/to/repo.git && git fetch -p origin && git push --mirror
  
- To *sync every 5 minutes*, use:
  sh
  */5 * * * * cd /path/to/repo.git && git fetch -p origin && git push --mirror
  

---

# *❓ Troubleshooting*
### *🔹 Git Permission Issues?*
Run:
sh
chmod 600 ~/.ssh/git_sync_key

If Git still asks for a password, ensure GitHub/GitLab *deploy keys* have been added.

### *🔹 Cron Job Not Running?*
- Check if cron service is running:
  sh
  systemctl status cron
  
  If not, start it:
  sh
  sudo systemctl start cron
  

---

# *🚀 Summary*
| Step | Description |
|------|------------|
| *1* | Install Git on your server |
| *2* | Generate SSH keys & add to GitHub/GitLab |
| *3* | Configure SSH for GitHub & GitLab |
| *4* | Clone the GitHub repo in mirror mode |
| *5* | Set GitLab as the push remote |
| *6* | Sync manually to test |
| *7* | Automate sync with a cron job |

---

This is the *most efficient free method* to mirror a *private GitHub repo to GitLab* 🚀.  

Let me know if you need *help debugging* or want *a script version*! 😊
