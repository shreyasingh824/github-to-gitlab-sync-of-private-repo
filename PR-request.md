[16:23, 29/01/2025] Shreya Singh Rathour: Here’s a *detailed step-by-step guide* to syncing a *private GitHub repository* to a *self-hosted GitLab* *for free* using a *cron job*.

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
sudo apt update && sudo apt install git -y   # Ub…
[17:17, 29/01/2025] Shreya Singh Rathour: Here's a *detailed step-by-step guide* to *fully migrate a private GitHub repository* to a *self-hosted GitLab* while preserving:  

✅ *Commits, branches, tags*  
✅ *Issues*  
✅ *Pull requests (PRs) → Merge Requests (MRs)*  
✅ *Wiki*  
✅ *Automated syncing*  

---

# *📌 Step 1: Prepare Authentication*
## *1.1 Generate an SSH Key for Git Authentication*
Since we're working with *private repositories, we need **SSH authentication*.  
Run the following command to generate an SSH key:  
sh
ssh-keygen -t rsa -b 4096 -C "git-sync" -f ~/.ssh/git_sync_key -N ""

- The SSH key is stored at: ~/.ssh/git_sync_key
- The *public key* (.pub file) will be needed later.

---

## *1.2 Add the SSH Key to GitHub & GitLab*
To access both repositories, add the *public key* (~/.ssh/git_sync_key.pub) to *both* GitHub and GitLab.  

### *➡️ Add SSH Key to GitHub*  
1. Go to *GitHub → Settings → SSH Keys*  
2. Click *"New SSH Key"*  
3. Copy & paste the contents of ~/.ssh/git_sync_key.pub  
4. Select *Read & Write* access  

### *➡️ Add SSH Key to GitLab*  
1. Go to *GitLab → Profile → SSH Keys*  
2. Click *"New SSH Key"*  
3. Copy & paste the same key (~/.ssh/git_sync_key.pub)  
4. Save  

---

## *1.3 Configure SSH for Git*
To avoid typing the *GitHub & GitLab hostnames* every time, create a config file:  
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

Then save (CTRL+X, Y, ENTER).

✅ *Now SSH authentication is set up!*  

---

# *📌 Step 2: Mirror the Git Repository*
## *2.1 Clone the GitHub Repository in Mirror Mode*
sh
mkdir -p /opt/github-to-gitlab-mirror
cd /opt/github-to-gitlab-mirror
git clone --mirror git@github.com:your-org/your-repo.git

- This will *download all commits, branches, and tags*.

---

## *2.2 Push to GitLab*
Now, *add GitLab as the remote repository* and *push everything*:
sh
cd /opt/github-to-gitlab-mirror/your-repo.git
git remote set-url --push origin git@gitlab.yourdomain.com:your-org/your-repo.git
git push --mirror

✅ *Your GitHub repository is now mirrored in GitLab!*  

---

# *📌 Step 3: Migrate Issues*
Since Git does *not track issues, we use the **GitHub API*.

## *3.1 Fetch Issues from GitHub*
sh
curl -H "Authorization: token YOUR_GITHUB_TOKEN" \
     -H "Accept: application/vnd.github.v3+json" \
     "https://api.github.com/repos/your-org/your-repo/issues" > github_issues.json

- This will create a JSON file with *all issues, labels, and comments*.

---

## *3.2 Create Issues in GitLab*
Now, loop through each issue and recreate it in GitLab:
sh
for i in $(jq -c '.[]' github_issues.json); do
  TITLE=$(echo $i | jq -r '.title')
  BODY=$(echo $i | jq -r '.body')

  curl --request POST --header "PRIVATE-TOKEN: YOUR_GITLAB_TOKEN" \
       --data "title=$TITLE&description=$BODY" \
       "https://gitlab.yourdomain.com/api/v4/projects/YOUR_PROJECT_ID/issues"
done

✅ *Issues are now transferred!*

---

# *📌 Step 4: Migrate Pull Requests (PRs)*
GitHub *PRs do not exist in Git*. Instead, we:
1. *Fetch PRs from GitHub*
2. *Create equivalent Merge Requests (MRs) in GitLab*

## *4.1 Fetch Pull Requests*
sh
curl -H "Authorization: token YOUR_GITHUB_TOKEN" \
     -H "Accept: application/vnd.github.v3+json" \
     "https://api.github.com/repos/your-org/your-repo/pulls" > github_prs.json

- This saves *open PRs* as JSON.

---

## *4.2 Create Merge Requests in GitLab*
sh
for i in $(jq -c '.[]' github_prs.json); do
  TITLE=$(echo $i | jq -r '.title')
  DESCRIPTION=$(echo $i | jq -r '.body')
  SOURCE_BRANCH=$(echo $i | jq -r '.head.ref')
  TARGET_BRANCH=$(echo $i | jq -r '.base.ref')

  curl --request POST --header "PRIVATE-TOKEN: YOUR_GITLAB_TOKEN" \
       --data "source_branch=$SOURCE_BRANCH&target_branch=$TARGET_BRANCH&title=$TITLE&description=$DESCRIPTION" \
       "https://gitlab.yourdomain.com/api/v4/projects/YOUR_PROJECT_ID/merge_requests"
done

✅ *PRs are now converted into GitLab Merge Requests!*

---

# *📌 Step 5: Migrate the Wiki*
GitHub wikis are stored in a *separate Git repository*.

## *5.1 Clone the Wiki from GitHub*
sh
git clone --mirror git@github.com:your-org/your-repo.wiki.git

## *5.2 Push to GitLab*
sh
cd your-repo.wiki.git
git remote set-url --push origin git@gitlab.yourdomain.com:your-org/your-repo.wiki.git
git push --mirror

✅ *Wiki is now migrated!*

---

# *📌 Step 6: Automate Syncing*
To keep *GitHub & GitLab in sync, set up a **cron job*.

## *6.1 Create a Sync Script*
Create a file:  
sh
nano /opt/github-to-gitlab-mirror/sync.sh

Paste this:
bash
#!/bin/bash
cd /opt/github-to-gitlab-mirror/your-repo.git
git fetch -p origin
git push --mirror

Make it executable:
sh
chmod +x /opt/github-to-gitlab-mirror/sync.sh

✅ *This script will sync changes from GitHub to GitLab.*

---

## *6.2 Schedule Sync Every Hour*
Run:
sh
crontab -e

Add this line:

0 * * * * /opt/github-to-gitlab-mirror/sync.sh

✅ *This will sync the repo every hour!*

---

# *📌 Final Summary*
| ✅ Step | 📌 Action |
|---------|----------|
| *1* | Setup SSH keys for authentication |
| *2* | Clone GitHub repo & push to GitLab |
| *3* | Fetch issues from GitHub & import them to GitLab |
| *4* | Fetch pull requests & create merge requests in GitLab |
| *5* | Sync GitHub Wiki to GitLab |
| *6* | Automate syncing with cron |

✅ *Now everything is migrated!* 🎉  
Let me know if you need any improvements or modifications! 🚀
