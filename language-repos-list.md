# Chloros Manual Language Repositories

This document lists all 28 language repositories for the Chloros manual translation project.

## Repository Naming Convention

`chloros_manual_gitbook-[language-code]`

## Repository List

| # | Language | Native Name | Repo Name | Language Code |
|---|----------|-------------|-----------|---------------|
| 1 | English | English | `chloros_manual_gitbook` | en (current) |
| 2 | Spanish | Español | `chloros_manual_gitbook-es` | es |
| 3 | Portuguese | Português | `chloros_manual_gitbook-pt` | pt |
| 4 | French | Français | `chloros_manual_gitbook-fr` | fr |
| 5 | German | Deutsch | `chloros_manual_gitbook-de` | de |
| 6 | Italian | Italiano | `chloros_manual_gitbook-it` | it |
| 7 | Japanese | 日本語 | `chloros_manual_gitbook-ja` | ja |
| 8 | Korean | 한국어 | `chloros_manual_gitbook-ko` | ko |
| 9 | Chinese (Simplified) | 简体中文 | `chloros_manual_gitbook-zh-CN` | zh-CN |
| 10 | Chinese (Traditional) | 繁體中文 | `chloros_manual_gitbook-zh-TW` | zh-TW |
| 11 | Russian | Русский | `chloros_manual_gitbook-ru` | ru |
| 12 | Dutch | Nederlands | `chloros_manual_gitbook-nl` | nl |
| 13 | Arabic | العربية | `chloros_manual_gitbook-ar` | ar |
| 14 | Polish | Polski | `chloros_manual_gitbook-pl` | pl |
| 15 | Turkish | Türkçe | `chloros_manual_gitbook-tr` | tr |
| 16 | Hindi | हिंदी | `chloros_manual_gitbook-hi` | hi |
| 17 | Indonesian | Bahasa Indonesia | `chloros_manual_gitbook-id` | id |
| 18 | Vietnamese | Tiếng Việt | `chloros_manual_gitbook-vi` | vi |
| 19 | Thai | ไทย | `chloros_manual_gitbook-th` | th |
| 20 | Swedish | Svenska | `chloros_manual_gitbook-sv` | sv |
| 21 | Danish | Dansk | `chloros_manual_gitbook-da` | da |
| 22 | Norwegian | Norsk | `chloros_manual_gitbook-no` | no |
| 23 | Finnish | Suomi | `chloros_manual_gitbook-fi` | fi |
| 24 | Greek | Ελληνικά | `chloros_manual_gitbook-el` | el |
| 25 | Czech | Čeština | `chloros_manual_gitbook-cs` | cs |
| 26 | Hungarian | Magyar | `chloros_manual_gitbook-hu` | hu |
| 27 | Romanian | Română | `chloros_manual_gitbook-ro` | ro |
| 28 | Ukrainian | Українська | `chloros_manual_gitbook-uk` | uk |

## GitBook Synchronization Plan

### Step 1: Create GitHub Repositories
Run the PowerShell script: `create-language-repos.ps1`

### Step 2: Duplicate GitBook Spaces
In GitBook:
1. Go to your current Chloros manual space
2. Duplicate the space 27 times
3. Name each space with the language (e.g., "Chloros Manual - Español", "Chloros Manual - 日本語", etc.)

### Step 3: Connect GitBook to GitHub
For each duplicated GitBook space:
1. Go to Space Settings → Integrations
2. Select "GitHub"
3. Connect to the corresponding language repository
4. Set up bi-directional sync
5. Choose the branch (typically `main`)

### Step 4: Translation
Once all GitBook spaces are synced:
1. Each space will have its content in the GitHub repo
2. Run translation scripts to translate markdown files
3. Changes will sync back to GitBook automatically

## Repository URLs (After Creation)

Assuming GitHub username is `MAPIR` or your organization:

- https://github.com/MAPIR/chloros_manual_gitbook (English - current)
- https://github.com/MAPIR/chloros_manual_gitbook-es
- https://github.com/MAPIR/chloros_manual_gitbook-pt
- https://github.com/MAPIR/chloros_manual_gitbook-fr
- https://github.com/MAPIR/chloros_manual_gitbook-de
- https://github.com/MAPIR/chloros_manual_gitbook-it
- https://github.com/MAPIR/chloros_manual_gitbook-ja
- https://github.com/MAPIR/chloros_manual_gitbook-ko
- https://github.com/MAPIR/chloros_manual_gitbook-zh-CN
- https://github.com/MAPIR/chloros_manual_gitbook-zh-TW
- https://github.com/MAPIR/chloros_manual_gitbook-ru
- https://github.com/MAPIR/chloros_manual_gitbook-nl
- https://github.com/MAPIR/chloros_manual_gitbook-ar
- https://github.com/MAPIR/chloros_manual_gitbook-pl
- https://github.com/MAPIR/chloros_manual_gitbook-tr
- https://github.com/MAPIR/chloros_manual_gitbook-hi
- https://github.com/MAPIR/chloros_manual_gitbook-id
- https://github.com/MAPIR/chloros_manual_gitbook-vi
- https://github.com/MAPIR/chloros_manual_gitbook-th
- https://github.com/MAPIR/chloros_manual_gitbook-sv
- https://github.com/MAPIR/chloros_manual_gitbook-da
- https://github.com/MAPIR/chloros_manual_gitbook-no
- https://github.com/MAPIR/chloros_manual_gitbook-fi
- https://github.com/MAPIR/chloros_manual_gitbook-el
- https://github.com/MAPIR/chloros_manual_gitbook-cs
- https://github.com/MAPIR/chloros_manual_gitbook-hu
- https://github.com/MAPIR/chloros_manual_gitbook-ro
- https://github.com/MAPIR/chloros_manual_gitbook-uk

## Translation Tools

After repository creation, additional scripts will be provided for:
- Automated markdown translation using AI/translation APIs
- Batch processing of all language repos
- Quality control and review workflows
- Maintaining sync between English (master) and translations

