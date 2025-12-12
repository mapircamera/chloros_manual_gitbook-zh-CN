# Chloros Manual Translation Project - Quick Start Guide

This guide provides a complete overview of setting up 27 language-specific repositories for the Chloros manual translation project.

## 📋 Overview

- **Total Languages**: 28 (including English)
- **New Repositories to Create**: 27 (English already exists)
- **Platform**: GitHub + GitBook
- **Workflow**: GitHub repos ↔️ GitBook spaces (bi-directional sync)

## 🚀 Quick Start (3 Steps)

### Step 1: Create GitHub Repositories

**Option A: Automated (Recommended)**
```powershell
# Run the PowerShell script
.\create-language-repos.ps1
```

**Option B: Manual**
- See `MANUAL-REPO-CREATION-INSTRUCTIONS.md` for detailed steps
- Use GitHub CLI or web interface to create each repo

### Step 2: Set Up GitBook Spaces

1. Log in to your GitBook account
2. Go to your existing "Chloros Manual" space
3. Duplicate the space 27 times:
   - Click the "..." menu → "Duplicate"
   - Name each: "Chloros Manual - [Language]"
   - Examples:
     - "Chloros Manual - Español"
     - "Chloros Manual - Français"
     - "Chloros Manual - 日本語"
     - etc.

### Step 3: Connect GitBook to GitHub

For each of the 27 new GitBook spaces:

1. Open the space
2. Go to **Space Settings** (gear icon)
3. Click **Integrations** → **GitHub**
4. Click **Connect** or **Configure**
5. Select the corresponding repository:
   - Spanish space → `chloros_manual_gitbook-es`
   - French space → `chloros_manual_gitbook-fr`
   - etc.
6. Choose sync direction: **Bi-directional** (recommended)
7. Select branch: **main**
8. Save settings
9. Wait for initial sync to complete

## 📊 Repository Mapping

| GitBook Space Name | GitHub Repository | Language Code |
|-------------------|-------------------|---------------|
| Chloros Manual - Español | chloros_manual_gitbook-es | es |
| Chloros Manual - Português | chloros_manual_gitbook-pt | pt |
| Chloros Manual - Français | chloros_manual_gitbook-fr | fr |
| Chloros Manual - Deutsch | chloros_manual_gitbook-de | de |
| Chloros Manual - Italiano | chloros_manual_gitbook-it | it |
| Chloros Manual - 日本語 | chloros_manual_gitbook-ja | ja |
| Chloros Manual - 한국어 | chloros_manual_gitbook-ko | ko |
| Chloros Manual - 简体中文 | chloros_manual_gitbook-zh-CN | zh-CN |
| Chloros Manual - 繁體中文 | chloros_manual_gitbook-zh-TW | zh-TW |
| Chloros Manual - Русский | chloros_manual_gitbook-ru | ru |
| Chloros Manual - Nederlands | chloros_manual_gitbook-nl | nl |
| Chloros Manual - العربية | chloros_manual_gitbook-ar | ar |
| Chloros Manual - Polski | chloros_manual_gitbook-pl | pl |
| Chloros Manual - Türkçe | chloros_manual_gitbook-tr | tr |
| Chloros Manual - हिंदी | chloros_manual_gitbook-hi | hi |
| Chloros Manual - Bahasa Indonesia | chloros_manual_gitbook-id | id |
| Chloros Manual - Tiếng Việt | chloros_manual_gitbook-vi | vi |
| Chloros Manual - ไทย | chloros_manual_gitbook-th | th |
| Chloros Manual - Svenska | chloros_manual_gitbook-sv | sv |
| Chloros Manual - Dansk | chloros_manual_gitbook-da | da |
| Chloros Manual - Norsk | chloros_manual_gitbook-no | no |
| Chloros Manual - Suomi | chloros_manual_gitbook-fi | fi |
| Chloros Manual - Ελληνικά | chloros_manual_gitbook-el | el |
| Chloros Manual - Čeština | chloros_manual_gitbook-cs | cs |
| Chloros Manual - Magyar | chloros_manual_gitbook-hu | hu |
| Chloros Manual - Română | chloros_manual_gitbook-ro | ro |
| Chloros Manual - Українська | chloros_manual_gitbook-uk | uk |

## 🔄 Workflow After Setup

Once all repos are created and synced:

### Phase 1: Initial Translation
1. Clone each language repository locally
2. Run AI translation scripts (to be provided)
3. Translate all `.md` files to target language
4. Commit and push changes
5. Changes automatically sync to GitBook

### Phase 2: Review & Refinement
1. Review translated content in GitBook interface
2. Make manual corrections as needed
3. Edit directly in GitBook or in GitHub
4. Changes sync automatically

### Phase 3: Maintenance
1. When English manual is updated, translate new content
2. Keep all language versions in sync
3. Use version control to track changes

## 📁 Files Created for This Project

- `create-language-repos.ps1` - Automated PowerShell script to create all repos
- `language-repos-list.md` - Complete list of all 28 language repositories
- `MANUAL-REPO-CREATION-INSTRUCTIONS.md` - Step-by-step manual creation guide
- `TRANSLATION-PROJECT-QUICKSTART.md` - This file

## 🛠️ Prerequisites

### Required
- GitHub account with repo creation permissions
- GitBook account with space duplication permissions
- Git installed on your computer

### Recommended
- GitHub CLI (`gh`) for automated repo creation
- PowerShell 5.1+ (Windows) for running automation scripts
- Code editor for reviewing translations

### Installing GitHub CLI

**Windows:**
```powershell
winget install --id GitHub.cli
```

Or download from: https://cli.github.com/

**After installation:**
```powershell
gh auth login
```

## 🎯 Success Checklist

- [ ] GitHub CLI installed and authenticated
- [ ] Ran `create-language-repos.ps1` or created repos manually
- [ ] Verified all 27 repos exist on GitHub
- [ ] Duplicated GitBook space 27 times
- [ ] Named each GitBook space appropriately
- [ ] Connected each GitBook space to corresponding GitHub repo
- [ ] Verified bi-directional sync is working
- [ ] Initial content synced from GitBook to GitHub
- [ ] Ready for translation phase

## 📞 Next Steps

After completing this setup:

1. **Notify the AI assistant**: Let them know all repos are created and synced
2. **Translation scripts**: Request AI-powered translation scripts
3. **Begin translation**: Start with high-priority languages
4. **Quality review**: Review translations for accuracy
5. **Publish**: Make translated manuals available to users

## 🌍 Language Priority (Suggested)

If translating in phases, consider this order based on global usage:

**Phase 1 (High Priority):**
- Spanish (es)
- Portuguese (pt)
- French (fr)
- German (de)
- Chinese Simplified (zh-CN)
- Japanese (ja)

**Phase 2 (Medium Priority):**
- Italian (it)
- Russian (ru)
- Korean (ko)
- Turkish (tr)
- Arabic (ar)
- Dutch (nl)

**Phase 3 (Complete Coverage):**
- All remaining languages

## 💡 Tips

1. **Batch Processing**: Process languages in small batches to maintain quality
2. **Native Review**: If possible, have native speakers review translations
3. **Consistency**: Use translation memory/glossaries for technical terms
4. **Version Control**: Tag releases in Git when major updates are complete
5. **Documentation**: Keep track of translation status in each repo's README

## 🐛 Troubleshooting

**Repos not creating?**
- Check GitHub CLI authentication: `gh auth status`
- Verify you have permission to create repos
- Check for rate limiting

**GitBook not syncing?**
- Verify GitHub integration is properly configured
- Check that the correct branch (main) is selected
- Ensure bi-directional sync is enabled
- Look for sync errors in GitBook settings

**Translation questions?**
- See language-specific translation guides (to be created)
- Consult Chloros terminology glossary (to be created)
- Contact project lead for clarification

## 📚 Additional Resources

- [GitBook Documentation](https://docs.gitbook.com/)
- [GitHub CLI Documentation](https://cli.github.com/manual/)
- [Git Documentation](https://git-scm.com/doc)
- [Chloros Main Site](https://www.mapir.camera)
- [Chloros Download](https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link)

---

**Ready to begin?** Run the PowerShell script or start creating repositories manually! 🚀

