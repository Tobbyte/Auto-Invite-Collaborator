# Auto Invite Collaborator

Automatically invite users to your repository via GitHub Issues. No more manual adding—just open an issue with a keyword and get invited instantly.

> Speak, friend, and enter

## What It Does

When someone opens an issue with the keyword **"invite me"** (or "Invite me"), this workflow:
1. ✅ Sends them a collaborator invitation
2. 📧 Posts a confirmation comment
3. 🔒 Closes the issue automatically

Perfect for educational projects.

---

## Setup (2 Steps)

### Step 1: Create a Fine-Grained Personal Access Token

1. Go to your GitHub profile → **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
2. Click **Generate new token**
3. Give it a name (e.g., "Invite Students Token")
4. Under **Repository access**, select your repository
5. Under **Permissions**, set:
   - **Administration**: Read and Write *(needed to invite collaborators)*
   - **Issues**: Read and Write *(needed to comment and close issues)*
6. Click **Generate token** and **copy it immediately**

### Step 2: Add Token to Repository Secrets

1. Go to your repository → **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `INVITE_TOKEN`
4. Paste your token into the Secret field
5. Click **Add secret**

---

## How to Use

1. Add the workflow file to `.github/workflows/auto-invite.yml` in your repository
2. Anyone can now open an issue with the title containing **"invite me"**
3. They'll automatically get invited with push permissions and receive a notification

---

## Customizing the Keyword

Edit the condition in the workflow file:

```yaml
if: contains(github.event.issue.title, 'your custom keyword') || contains(github.event.issue.title, 'Your Custom Keyword')
```

---

## Workflow File

Place this in `.github/workflows/auto-invite.yml`:

```yaml
name: Auto Invite Collaborator

on:
  issues:
    types: [opened, edited]

jobs:
  invite:
    runs-on: ubuntu-latest
    if: contains(github.event.issue.title, 'invite me') || contains(github.event.issue.title, 'Invite me')
    
    steps:
      - name: Invite User via GitHub CLI
        run: |
          gh api \
            --method PUT \
            -H "Accept: application/vnd.github+json" \
            /repos/${{ github.repository }}/collaborators/${{ github.event.issue.user.login }} \
            -f permission='push'
        env:
          GH_TOKEN: ${{ secrets.INVITE_TOKEN }}

      - name: Close the issue with a comment
        run: |
          gh issue comment ${{ github.event.issue.number }} --body "Hi @${{ github.event.issue.user.login }}! 👋 An invitation has been sent to your GitHub account and email. Please check your notifications or email to accept it and start collaborating!"
          gh issue close ${{ github.event.issue.number }}
        env:
          GH_TOKEN: ${{ secrets.INVITE_TOKEN }}
```
---

## ToDo:
- check if issue could be deleted (instead of closed) to keep passphrase "secret"

---

## Security Notes

- The fine-grained token is scoped to **only this repository**
- Grants permissions for **administration**, so be careful, there'll be 🐉

---

## License

MIT