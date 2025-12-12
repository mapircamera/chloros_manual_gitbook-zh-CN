# Chloros Manual Translation Project

## 🌍 Project Overview

This project sets up infrastructure for translating the Chloros Professional Multispectral Image Processing Software manual into **37 additional languages** (38 total including English), matching the language support in the Chloros application itself.

### Project Goals
- Create separate GitHub repositories for each language translation
- Sync each repository with a dedicated GitBook space
- Enable efficient translation and maintenance workflows
- Provide users worldwide with documentation in their native language

## 📊 Project Statistics

- **Total Languages**: 38
- **New Repositories**: 37
- **Existing Repository**: 1 (English - current)
- **Total GitBook Spaces**: 38 (1 existing + 37 new)
- **Documentation Files**: ~15 markdown files per language (estimated)

## 🗂️ Project Files

This directory contains the following files to help you set up the translation project:

### 1. **create-language-repos.ps1** ⚡
**Automated PowerShell script** that creates all 27 GitHub repositories in one go.
- Interactive with confirmation prompts
- Creates repos with proper naming and descriptions
- Initializes each repo with README and .gitignore
- Makes initial commits and pushes to GitHub
- Provides success/failure summary

**Usage:**
```powershell
.\create-language-repos.ps1
```

### 2. **language-repos-list.md** 📋
**Complete reference document** listing all 28 language repositories.
- Table with all languages and repo names
- Language codes for each translation
- Repository naming convention
- URL format for all repos
- GitBook synchronization plan

### 3. **MANUAL-REPO-CREATION-INSTRUCTIONS.md** 📖
**Step-by-step guide** for creating repositories manually.
- Instructions for GitHub CLI method
- Instructions for GitHub web interface method
- Full command list for quick copy-paste
- Troubleshooting section
- Alternative approaches if automation doesn't work

### 4. **TRANSLATION-PROJECT-QUICKSTART.md** 🚀
**Quick start guide** with 3-step setup process.
- Overview of entire workflow
- Repository mapping table
- GitBook connection instructions
- Success checklist
- Translation phase guidelines
- Suggested language priority order

### 5. **create-repos-commands.txt** 📝
**Simple text file** with all GitHub CLI commands.
- One command per language repository
- Ready to copy and paste
- No scripting knowledge required
- Can be run individually or all at once

### 6. **TRANSLATION-PROJECT-README.md** (this file) 📚
**Master documentation** that explains the entire project.
- Project overview and structure
- File descriptions
- Workflow summary
- Next steps

## 🎯 Complete Setup Workflow

### Phase 1: Repository Creation (GitHub)
**Choose one method:**

**Method A - Automated (Recommended):**
```powershell
.\create-language-repos.ps1
```

**Method B - Manual with GitHub CLI:**
```powershell
# Copy commands from create-repos-commands.txt
# Paste into PowerShell/Terminal
```

**Method C - Manual via Web:**
- Follow instructions in `MANUAL-REPO-CREATION-INSTRUCTIONS.md`
- Create each repo through GitHub web interface

### Phase 2: GitBook Space Creation
1. Log into GitBook
2. Navigate to existing "Chloros Manual" space
3. Duplicate space 27 times
4. Name each space: "Chloros Manual - [Native Language Name]"
   - Example: "Chloros Manual - Español"
   - Example: "Chloros Manual - 日本語"

### Phase 3: Connect GitBook to GitHub
For each of the 27 new GitBook spaces:
1. Open space settings
2. Go to Integrations → GitHub
3. Connect to corresponding repository
4. Enable bi-directional sync
5. Select `main` branch
6. Save and wait for initial sync

### Phase 4: Translation
1. Clone language repositories locally
2. Run AI translation tools (to be provided after setup)
3. Review and refine translations
4. Commit changes to GitHub
5. Changes automatically sync to GitBook

## 🌐 All 38 Languages

| # | Language | Native Name | Code | Repo Name |
|---|----------|-------------|------|-----------|
| 1 | English | English | en | chloros_manual_gitbook (current) |
| 2 | Spanish | Español | es | chloros_manual_gitbook-es |
| 3 | Portuguese | Português | pt | chloros_manual_gitbook-pt |
| 4 | French | Français | fr | chloros_manual_gitbook-fr |
| 5 | German | Deutsch | de | chloros_manual_gitbook-de |
| 6 | Italian | Italiano | it | chloros_manual_gitbook-it |
| 7 | Japanese | 日本語 | ja | chloros_manual_gitbook-ja |
| 8 | Korean | 한국어 | ko | chloros_manual_gitbook-ko |
| 9 | Chinese (S) | 简体中文 | zh-CN | chloros_manual_gitbook-zh-CN |
| 10 | Chinese (T) | 繁體中文 | zh-TW | chloros_manual_gitbook-zh-TW |
| 11 | Russian | Русский | ru | chloros_manual_gitbook-ru |
| 12 | Dutch | Nederlands | nl | chloros_manual_gitbook-nl |
| 13 | Arabic | العربية | ar | chloros_manual_gitbook-ar |
| 14 | Polish | Polski | pl | chloros_manual_gitbook-pl |
| 15 | Turkish | Türkçe | tr | chloros_manual_gitbook-tr |
| 16 | Hindi | हिंदी | hi | chloros_manual_gitbook-hi |
| 17 | Indonesian | Bahasa Indonesia | id | chloros_manual_gitbook-id |
| 18 | Vietnamese | Tiếng Việt | vi | chloros_manual_gitbook-vi |
| 19 | Thai | ไทย | th | chloros_manual_gitbook-th |
| 20 | Swedish | Svenska | sv | chloros_manual_gitbook-sv |
| 21 | Danish | Dansk | da | chloros_manual_gitbook-da |
| 22 | Norwegian | Norsk | no | chloros_manual_gitbook-no |
| 23 | Finnish | Suomi | fi | chloros_manual_gitbook-fi |
| 24 | Greek | Ελληνικά | el | chloros_manual_gitbook-el |
| 25 | Czech | Čeština | cs | chloros_manual_gitbook-cs |
| 26 | Hungarian | Magyar | hu | chloros_manual_gitbook-hu |
| 27 | Romanian | Română | ro | chloros_manual_gitbook-ro |
| 28 | Ukrainian | Українська | uk | chloros_manual_gitbook-uk |
| 29 | Brazilian Portuguese | Português Brasileiro | pt-BR | chloros_manual_gitbook-pt-BR |
| 30 | Cantonese | 粵語 | zh-HK | chloros_manual_gitbook-zh-HK |
| 31 | Malay | Bahasa Melayu | ms | chloros_manual_gitbook-ms |
| 32 | Slovak | Slovenčina | sk | chloros_manual_gitbook-sk |
| 33 | Bulgarian | Български | bg | chloros_manual_gitbook-bg |
| 34 | Croatian | Hrvatski | hr | chloros_manual_gitbook-hr |
| 35 | Lithuanian | Lietuvių | lt | chloros_manual_gitbook-lt |
| 36 | Latvian | Latviešu | lv | chloros_manual_gitbook-lv |
| 37 | Estonian | Eesti | et | chloros_manual_gitbook-et |
| 38 | Slovenian | Slovenščina | sl | chloros_manual_gitbook-sl |

## 🔧 Prerequisites

### Required Tools
- **Git**: Version control system
  - Download: https://git-scm.com/
- **GitHub Account**: With repository creation permissions
  - Sign up: https://github.com/
- **GitBook Account**: With space management permissions
  - Sign up: https://www.gitbook.com/

### Recommended Tools
- **GitHub CLI** (`gh`): For automated repo creation
  - Windows: `winget install --id GitHub.cli`
  - Download: https://cli.github.com/
- **PowerShell 5.1+**: For running automation scripts (Windows)
- **Code Editor**: VS Code, Sublime Text, or similar

### Authentication
After installing GitHub CLI:
```powershell
gh auth login
```
Follow the prompts to authenticate with your GitHub account.

## 📁 Current Repository Structure

```
chloros_manual_gitbook/
├── api-python-sdk.md
├── calibration-targets.md
├── chloros-product-page.html
├── chloros+-login.md
├── CLI.md
├── download.md
├── faq.md
├── image-viewer-gui/
│   ├── image-layers.md
│   ├── index-lut-sandbox.md
│   └── page-3.md
├── navigation.md
├── output-image-formats.md
├── processing-images-gui/
│   ├── adjusting-project-settings.md
│   ├── choosing-target-images.md
│   ├── finishing-the-processing.md
│   ├── monitoring-the-processing.md
│   ├── page-1.md
│   └── starting-the-processing.md
├── project-settings/
│   ├── multispectral-index-formulas.md
│   └── page-2.md
├── projects.md
├── README.md
├── SUMMARY.md
├── supported-cameras.md
└── supported-languages.md
```

This entire structure will be duplicated to each language repository and then translated.

## 🚀 Quick Start (TL;DR)

1. **Install GitHub CLI**: `winget install --id GitHub.cli`
2. **Authenticate**: `gh auth login`
3. **Run Script**: `.\create-language-repos.ps1`
4. **Create GitBook Spaces**: Duplicate your space 27 times
5. **Connect**: Link each GitBook space to its GitHub repo
6. **Translate**: Use AI tools to translate content (next phase)

## ✅ Success Criteria

You'll know the setup is complete when:
- ✅ All 27 GitHub repositories exist
- ✅ All 27 GitBook spaces are created
- ✅ Each GitBook space is connected to its GitHub repo
- ✅ Bi-directional sync is enabled
- ✅ Initial content is synced from GitBook to GitHub
- ✅ You can edit in either GitBook or GitHub and see changes sync

## 🔄 Translation Workflow (Next Phase)

After setup is complete:

1. **Clone Repos**: Clone all language repos locally
2. **Prepare Translation Tools**: Set up AI translation scripts
3. **Translate Content**: Run translation scripts on each repo
4. **Review**: Check translations for accuracy
5. **Refine**: Make manual corrections as needed
6. **Commit & Push**: Push translations to GitHub
7. **Auto-Sync**: Changes appear in GitBook automatically
8. **Publish**: Make translated manuals available to users

## 📞 Support & Resources

- **Chloros Website**: https://www.mapir.camera
- **Chloros Download**: https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link
- **Chloros+ Plans**: https://cloud.mapir.camera/pricing
- **English Manual**: https://mapir.gitbook.io/chloros
- **Contact**: info@mapir.camera

## 🤝 Contributing

After initial setup and translation:
- Native speakers are welcome to review and improve translations
- Submit pull requests to language-specific repositories
- Report translation issues via GitHub Issues
- Suggest terminology improvements

## 📝 Notes

- **Naming Convention**: `chloros_manual_gitbook-[language-code]`
- **Branch Strategy**: Use `main` branch for all repos
- **Sync Direction**: Bi-directional (GitHub ↔️ GitBook)
- **Update Strategy**: Update English first, then propagate to translations
- **Version Control**: Tag major releases in Git

## 🎉 Let's Get Started!

Ready to create the repositories? Choose your preferred method from the workflow above and begin! If you run into any issues, consult the detailed guides in this directory.

---

**Questions?** Contact MAPIR at info@mapir.camera or visit https://www.mapir.camera/community/contact

