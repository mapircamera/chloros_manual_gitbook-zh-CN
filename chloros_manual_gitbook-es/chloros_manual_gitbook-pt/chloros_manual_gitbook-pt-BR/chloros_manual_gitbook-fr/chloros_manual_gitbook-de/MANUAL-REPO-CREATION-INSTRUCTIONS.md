# Manual Repository Creation Instructions

If you prefer to create repositories manually or if the automated script doesn't work, follow these instructions.

## Prerequisites

1. GitHub account with appropriate permissions
2. Git installed on your computer
3. GitHub CLI (optional but recommended): https://cli.github.com/

## Method 1: Using GitHub CLI (Recommended)

### Install GitHub CLI
```powershell
# Windows (using winget)
winget install --id GitHub.cli

# Or download from https://cli.github.com/
```

### Authenticate
```powershell
gh auth login
```

### Create Each Repository
For each language, run:

```powershell
# Example for Spanish
gh repo create chloros_manual_gitbook-es --public --description "Chloros Professional Multispectral Image Processing Software - User Manual (Español)" --clone

cd chloros_manual_gitbook-es

# Create README.md
@"
# Chloros User Manual - Español

This is the Spanish translation of the Chloros Professional Multispectral Image Processing Software user manual.

## About Chloros
Chloros is professional multispectral image processing software that transforms imagery into actionable insights.

## Links
- [Main Website](https://www.mapir.camera)
- [Chloros Download](https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link)
- [English Documentation](https://mapir.gitbook.io/chloros)
"@ | Out-File README.md -Encoding utf8

# Create .gitignore
@"
.DS_Store
Thumbs.db
.vscode/
_book/
node_modules/
"@ | Out-File .gitignore -Encoding utf8

# Commit and push
git add .
git commit -m "Initial commit - Spanish translation repository"
git push -u origin main

cd ..
```

Repeat for all 27 languages (see language-repos-list.md for all language codes).

## Method 2: Using GitHub Web Interface

### For Each Language:

1. **Go to GitHub**
   - Navigate to https://github.com/new
   
2. **Repository Settings**
   - Repository name: `chloros_manual_gitbook-[language-code]`
   - Description: `Chloros Professional Multispectral Image Processing Software - User Manual ([Native Language Name])`
   - Public repository
   - ✓ Add a README file
   - Add .gitignore: None (we'll add it later)
   - License: None (or your preferred license)

3. **Click "Create repository"**

4. **Clone Locally**
   ```powershell
   git clone https://github.com/YOUR-USERNAME/chloros_manual_gitbook-[language-code]
   cd chloros_manual_gitbook-[language-code]
   ```

5. **Add Files**
   - Edit README.md with appropriate content
   - Create .gitignore file
   - Commit and push

6. **Repeat** for all 27 languages

## All Repository Names to Create

Copy and paste this list for reference:

```
chloros_manual_gitbook-es       (Spanish - Español)
chloros_manual_gitbook-pt       (Portuguese - Português)
chloros_manual_gitbook-fr       (French - Français)
chloros_manual_gitbook-de       (German - Deutsch)
chloros_manual_gitbook-it       (Italian - Italiano)
chloros_manual_gitbook-ja       (Japanese - 日本語)
chloros_manual_gitbook-ko       (Korean - 한국어)
chloros_manual_gitbook-zh-CN    (Chinese Simplified - 简体中文)
chloros_manual_gitbook-zh-TW    (Chinese Traditional - 繁體中文)
chloros_manual_gitbook-ru       (Russian - Русский)
chloros_manual_gitbook-nl       (Dutch - Nederlands)
chloros_manual_gitbook-ar       (Arabic - العربية)
chloros_manual_gitbook-pl       (Polish - Polski)
chloros_manual_gitbook-tr       (Turkish - Türkçe)
chloros_manual_gitbook-hi       (Hindi - हिंदी)
chloros_manual_gitbook-id       (Indonesian - Bahasa Indonesia)
chloros_manual_gitbook-vi       (Vietnamese - Tiếng Việt)
chloros_manual_gitbook-th       (Thai - ไทย)
chloros_manual_gitbook-sv       (Swedish - Svenska)
chloros_manual_gitbook-da       (Danish - Dansk)
chloros_manual_gitbook-no       (Norwegian - Norsk)
chloros_manual_gitbook-fi       (Finnish - Suomi)
chloros_manual_gitbook-el       (Greek - Ελληνικά)
chloros_manual_gitbook-cs       (Czech - Čeština)
chloros_manual_gitbook-hu       (Hungarian - Magyar)
chloros_manual_gitbook-ro       (Romanian - Română)
chloros_manual_gitbook-uk       (Ukrainian - Українська)
```

## Quick Command List (GitHub CLI)

If you have GitHub CLI installed, you can run these commands in sequence:

```powershell
gh repo create chloros_manual_gitbook-es --public --description "Chloros User Manual - Español" --clone
gh repo create chloros_manual_gitbook-pt --public --description "Chloros User Manual - Português" --clone
gh repo create chloros_manual_gitbook-fr --public --description "Chloros User Manual - Français" --clone
gh repo create chloros_manual_gitbook-de --public --description "Chloros User Manual - Deutsch" --clone
gh repo create chloros_manual_gitbook-it --public --description "Chloros User Manual - Italiano" --clone
gh repo create chloros_manual_gitbook-ja --public --description "Chloros User Manual - 日本語" --clone
gh repo create chloros_manual_gitbook-ko --public --description "Chloros User Manual - 한국어" --clone
gh repo create chloros_manual_gitbook-zh-CN --public --description "Chloros User Manual - 简体中文" --clone
gh repo create chloros_manual_gitbook-zh-TW --public --description "Chloros User Manual - 繁體中文" --clone
gh repo create chloros_manual_gitbook-ru --public --description "Chloros User Manual - Русский" --clone
gh repo create chloros_manual_gitbook-nl --public --description "Chloros User Manual - Nederlands" --clone
gh repo create chloros_manual_gitbook-ar --public --description "Chloros User Manual - العربية" --clone
gh repo create chloros_manual_gitbook-pl --public --description "Chloros User Manual - Polski" --clone
gh repo create chloros_manual_gitbook-tr --public --description "Chloros User Manual - Türkçe" --clone
gh repo create chloros_manual_gitbook-hi --public --description "Chloros User Manual - हिंदी" --clone
gh repo create chloros_manual_gitbook-id --public --description "Chloros User Manual - Bahasa Indonesia" --clone
gh repo create chloros_manual_gitbook-vi --public --description "Chloros User Manual - Tiếng Việt" --clone
gh repo create chloros_manual_gitbook-th --public --description "Chloros User Manual - ไทย" --clone
gh repo create chloros_manual_gitbook-sv --public --description "Chloros User Manual - Svenska" --clone
gh repo create chloros_manual_gitbook-da --public --description "Chloros User Manual - Dansk" --clone
gh repo create chloros_manual_gitbook-no --public --description "Chloros User Manual - Norsk" --clone
gh repo create chloros_manual_gitbook-fi --public --description "Chloros User Manual - Suomi" --clone
gh repo create chloros_manual_gitbook-el --public --description "Chloros User Manual - Ελληνικά" --clone
gh repo create chloros_manual_gitbook-cs --public --description "Chloros User Manual - Čeština" --clone
gh repo create chloros_manual_gitbook-hu --public --description "Chloros User Manual - Magyar" --clone
gh repo create chloros_manual_gitbook-ro --public --description "Chloros User Manual - Română" --clone
gh repo create chloros_manual_gitbook-uk --public --description "Chloros User Manual - Українська" --clone
```

## Next Steps After Repo Creation

1. **In GitBook**: Duplicate your Chloros manual 27 times (one for each language)
2. **Name each space**: "Chloros Manual - [Native Language Name]"
3. **Connect to GitHub**: In each GitBook space settings, connect to the corresponding GitHub repo
4. **Enable Sync**: Set up bi-directional sync between GitBook and GitHub
5. **Verify**: Check that all repos are properly synced
6. **Translate**: Use translation tools to translate the content in each repo

## Troubleshooting

### Issue: "Repository already exists"
- The repository name is already taken
- Choose a different name or delete the existing repo first

### Issue: GitHub CLI not authenticated
Run: `gh auth login`

### Issue: Permission denied
- Make sure you have permission to create repos in your account/organization
- Check your GitHub token permissions

### Issue: Rate limiting
- GitHub has API rate limits
- Wait a few minutes between creating repositories
- Or create them manually through the web interface

