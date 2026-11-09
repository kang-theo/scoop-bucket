# My Scoop Bucket

Personal [Scoop](https://scoop.sh/) Bucket for managing Windows applications.

| Application | Type | Version |
| --- | --- | --- |
| [VimDesktop](https://github.com/goreliu/vimdesktop) | GUI | 2.2.2 |
| [TUIOS](https://github.com/Gaurav-Gosain/tuios) | CLI | 0.12.1 |

## 1. Create a Bucket

1. Open [Scoop BucketTemplate](https://github.com/ScoopInstaller/BucketTemplate).
2. Click **Use this template → Create a new repository**, named `scoop-bucket`.
3. Configure **Settings → Actions → General**:
   - **Actions permissions:** Allow all actions and reusable workflows.
   - **Workflow permissions:** Read and write permissions.
   - Enable Actions to create pull requests if required.
4. Replace the repository placeholder in `bin/auto-pr.ps1`.

Clone the repository:

```powershell
git clone https://github.com/<username>/scoop-bucket.git
cd scoop-bucket
```

<details>
<summary>Repository structure</summary>

```text
scoop-bucket/
├── bucket/
│   ├── vimdesktop.json
│   └── tuios.json
├── .github/workflows/
├── bin/
│   └── auto-pr.ps1
└── README.md
```

</details>

## 2. Create Application Manifests

Create one JSON file per application in `bucket/`. The following examples include automatic version-checking rules.

<details>
<summary><strong>VimDesktop — bucket/vimdesktop.json</strong></summary>

Release: [VimDesktop v2.2.2](https://github.com/goreliu/vimdesktop/releases/tag/v2.2.2)

```json
{
  "version": "2.2.2",
  "description": "Vim-style keyboard navigation for Windows",
  "homepage": "https://github.com/goreliu/vimdesktop",
  "architecture": {
    "64bit": {
      "url": "https://github.com/goreliu/vimdesktop/releases/download/v2.2.2/VimDesktop-x86_64-2.2.2.7z",
      "hash": "<SHA256>"
    }
  },
  "checkver": "github",
  "autoupdate": {
    "architecture": {
      "64bit": {
        "url": "https://github.com/goreliu/vimdesktop/releases/download/v$version/VimDesktop-x86_64-$version.7z"
      }
    }
  }
}
```

</details>

<details>
<summary><strong>TUIOS — bucket/tuios.json</strong></summary>

Release: [TUIOS v0.12.1](https://github.com/Gaurav-Gosain/tuios/releases/tag/v0.12.1)

```json
{
  "version": "0.12.1",
  "description": "Terminal window manager with tiling panes and persistent sessions",
  "homepage": "https://github.com/Gaurav-Gosain/tuios",
  "license": "MIT",
  "architecture": {
    "64bit": {
      "url": "https://github.com/Gaurav-Gosain/tuios/releases/download/v0.12.1/tuios_0.12.1_Windows_x86_64.tar.gz",
      "hash": "<SHA256>"
    }
  },
  "bin": "tuios.exe",
  "checkver": "github",
  "autoupdate": {
    "architecture": {
      "64bit": {
        "url": "https://github.com/Gaurav-Gosain/tuios/releases/download/v$version/tuios_$version_Windows_x86_64.tar.gz"
      }
    }
  }
}
```

</details>

### Generate SHA256

Download each release archive and calculate its checksum:

```powershell
(Get-FileHash .\downloaded-file.7z -Algorithm SHA256).Hash.ToLower()
```

Replace `<SHA256>` with the actual checksum. Use the `.tar.gz` filename for TUIOS.

<details>
<summary>Manifest fields explained</summary>

| Field | Purpose |
| --- | --- |
| `version` | Application version |
| `url` | Download URL |
| `hash` | SHA256 verification |
| `architecture` | Supported CPU architectures |
| `bin` | Register terminal commands |
| `shortcuts` | Create Start Menu shortcuts |
| `persist` | Preserve configuration across updates |
| `checkver` | Detect new releases |
| `autoupdate` | Generate version-specific URLs |

Scoop supports `.7z` and `.tar.gz` archives. Verify executable paths inside archives before configuring `bin` or `shortcuts`.

</details>

## 3. Publish and Install

Commit and push the Manifests:

```powershell
git add .
git commit -m "add vimdesktop and tuios"
git push
```

Add the Bucket to Scoop (once per computer):

```powershell
scoop bucket add myapps https://github.com/<username>/scoop-bucket
```

Install:

```powershell
scoop install myapps/vimdesktop
scoop install myapps/tuios
```

<details>
<summary>Common Scoop commands</summary>

| Command | Description |
| --- | --- |
| `scoop bucket list` | List Buckets |
| `scoop search tuios` | Search applications |
| `scoop cat myapps/tuios` | View Manifest |
| `scoop update` | Refresh Scoop and Buckets |
| `scoop update *` | Update installed applications |
| `scoop uninstall tuios` | Uninstall application |
| `scoop bucket rm myapps` | Remove Bucket |

</details>

## 4. Automatic Updates

**A. Update Bucket Manifests**

`checkver` detects new GitHub Releases; `autoupdate` defines the URL pattern for generating updated Manifests and hashes.

Run from the repository root:

```powershell
$checkver = "$HOME\scoop\apps\scoop\current\bin\checkver.ps1"

& $checkver -App vimdesktop -Dir .\bucket -Update
& $checkver -App tuios -Dir .\bucket -Update
```

Review and publish:

```powershell
git diff
git add bucket
git commit -m "update application versions"
git push
```

Alternatively, configure the BucketTemplate GitHub Actions to detect new releases and create update pull requests.

**B. Update Installed Applications**

After updated Manifests have been published:

```powershell
scoop update
scoop update *
```

<details>
<summary>How automatic updates work</summary>

```text
GitHub Releases
      ↓
checkver + autoupdate
      ↓
Update Manifest (version, URL, SHA256)
      ↓
GitHub Actions → Pull Request → Merge
      ↓
scoop update
scoop update *
```

- `checkver` detects new versions.
- `autoupdate` provides the new download URL pattern.
- GitHub Actions can automate Manifest maintenance.
- Scoop updates installed applications using published Manifests.
- `autoupdate` alone does not schedule or install updates.

</details>

## 5. Notes

<details>
<summary>Configuration and troubleshooting</summary>

- **GUI vs CLI:** VimDesktop can use `shortcuts`; TUIOS uses `bin`.
- **Configuration:** Use `persist` if application settings reside inside the installation directory.
- **Executable paths:** Check archive contents before configuring `bin` or `shortcuts`.
- **SHA256 errors:** Recalculate the hash from the exact release archive.
- **Version patterns:** Update `autoupdate` if upstream release filenames change.
- **Built-in updaters:** Avoid mixing Scoop updates with applications' own update mechanisms.

</details>

## References

[Scoop](https://scoop.sh/) · [BucketTemplate](https://github.com/ScoopInstaller/BucketTemplate) · [App Manifests](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests)
