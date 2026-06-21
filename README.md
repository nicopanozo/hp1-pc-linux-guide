# Running Harry Potter and the Philosopher's Stone (PC, 2001) on Ubuntu 26.04 with Wine

**Date written:** June 21, 2026  
**Tested on:** Ubuntu 26.04 "Resolute" — Dell XPS 15 9570  
**Monitor:** 3440×1440 ultrawide (21:9)  
**Author's note:** This is a personal write-up of the exact steps I followed to get this game running on my Linux machine. It is not perfect — there are likely better ways to do some of this — but it worked, and I'm sharing it so others don't have to spend hours figuring it out. If you find improvements, please share them with the community.

---

## Table of Contents

1. [Background and Context](#background-and-context)
2. [What You'll Need](#what-youll-need)
3. [What We Learned About This Game's Technical Challenges](#what-we-learned-about-this-games-technical-challenges)
4. [Step 1 — Download the Game](#step-1--download-the-game)
5. [Step 2 — Install Lutris and Wine](#step-2--install-lutris-and-wine)
6. [Step 3 — Mount the ISO](#step-3--mount-the-iso)
7. [Step 4 — Install the Game via Wine](#step-4--install-the-game-via-wine)
8. [Step 5 — Fix the Missing System Files](#step-5--fix-the-missing-system-files)
9. [Step 6 — Download the DRM-Free Replacement Executable](#step-6--download-the-drm-free-replacement-executable)
10. [Step 7 — Download and Apply Modern Fixes](#step-7--download-and-apply-modern-fixes)
11. [Step 8 — Copy All Game Assets from the ISO](#step-8--copy-all-game-assets-from-the-iso)
12. [Step 9 — Add the Game to Lutris](#step-9--add-the-game-to-lutris)
13. [Step 10 — Configure the Game (Resolution + Language)](#step-10--configure-the-game-resolution--language)
14. [Step 11 — Launch and Play](#step-11--launch-and-play)
15. [Spanish Language Notes](#spanish-language-notes)
16. [Known Issues and Limitations](#known-issues-and-limitations)
17. [Key Findings and Diagnostic Tricks](#key-findings-and-diagnostic-tricks)
18. [Where to Share and Legal Notes](#where-to-share-and-legal-notes)

---

## Background and Context

Harry Potter and the Philosopher's Stone (also released as "Sorcerer's Stone" in the US) was released in 2001 by EA Games, built on the Unreal Engine. It is a classic 3D adventure game that has not been officially re-released on any modern platform. The only way to play it today is through the original PC disc or an abandonware source.

The main obstacle on modern systems (Windows 10+, Linux) is its copy protection: **SafeDisc (secdrv.sys)**. This driver-level DRM was removed from modern Windows and simply does not run on Linux at all. Without replacing the copy-protected executable, the game refuses to launch entirely.

This guide documents how to run it on **Ubuntu 26.04 "Resolute"** using Wine and Lutris, with modern compatibility fixes applied.

---

## What You'll Need

- Ubuntu 26.04 (should work on other recent Ubuntu versions too)
- Internet connection
- About 2 GB of free disk space
- The game ISO from MyAbandonware (see Step 1)
- The open-source DRM-free replacement HP.exe from Internet Archive (see Step 6)
- Several community-made fix files from MyAbandonware (see Step 7)

---

## What We Learned About This Game's Technical Challenges

Before diving in, here's a summary of the problems you'll hit and why:

| Problem | Cause | Solution |
|---|---|---|
| Game won't launch at all | SafeDisc DRM (secdrv.sys) incompatible with modern OS | Replace HP.exe with open-source DRM-free replacement executable |
| `c0000135` error | Wine can't find the executable (wrong filename or missing 32-bit support) | The actual executable is `HP.exe`, not `HarryPotter.exe` |
| Game window tiny (top-left corner) | Default resolution is 640×480 | Edit HP.ini to set resolution and windowed mode |
| Game in English despite Spanish files | `Language=int` not changed to correct code | Set `Language=spa` in both HP.ini and Default.ini |
| Voices still in English after text fix | US version only has English audio | Cannot be fixed without a different regional disc |
| Missing files after install | Installer doesn't copy everything from the disc | Manually copy System, Maps, Sounds, Textures, Music folders |

---

## Step 1 — Download the Game

Go to **[myabandonware.com](https://www.myabandonware.com)** and search for:

> Harry Potter and the Sorcerer's Stone

Download the **"US version (English & Spanish languages)"** — this version is approximately 354 MB and comes as a ZIP file containing an ISO disc image.

> **Note:** The site also offers a "MagiPack Repack" which is smaller, pre-patched, and does not require a product key during installation, but it is English-only. If you don't care about Spanish text, that version is easier to get running. This guide uses the US version.

During installation you will need a **product key**. Do not post product keys in this repo or in public comments — community members share them in the MyAbandonware download page comments.

Extract the ZIP. Inside you'll find a single file: `HARRY_POTTER_US.iso`

---

## Step 2 — Install Lutris and Wine

**Important note about Ubuntu 26.04:** The Lutris PPA (`ppa:lutris-team/lutris`) does **not** have a release file for Ubuntu 26.04 "Resolute" yet as of June 2026. You will see a 404 error when adding it. This is fine — Ubuntu's own repositories already include a recent version of Lutris, so you can ignore the PPA error and proceed.

```bash
sudo add-apt-repository ppa:lutris-team/lutris   # Will show 404 error — that's OK
sudo apt update
sudo apt install lutris
```

Lutris will automatically pull in Wine and all necessary dependencies, including `wine`, `wine64`, `winetricks`, `gamescope`, and many others (about 37 packages, ~217 MB).

Also add 32-bit architecture support (even though wine32 may already be installed, this ensures nothing is missing):

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine32:i386
```

---

## Step 3 — Mount the ISO

```bash
sudo mkdir -p /mnt/hp1
sudo mount -o loop ~/Downloads/HP1/HARRY_POTTER_US.iso /mnt/hp1
```

You'll see a message saying it was mounted read-only — that's normal and expected for an ISO.

Verify the contents:

```bash
ls /mnt/hp1
```

You should see: `AutoRun.exe`, `DirectX`, `Help`, `Maps`, `Music`, `ReadMe`, `Sounds`, `Support`, `System`, `Textures`, and a few other files.

> **After a reboot:** The ISO will be unmounted. You'll need to re-run the mount command if you restart your computer mid-setup.

---

## Step 4 — Install the Game via Wine

```bash
wine /mnt/hp1/setup/Setup.exe
```

A Windows-style installer window will appear (you can ignore the Wine error messages in the terminal — they are harmless). Go through the installation wizard. When prompted for a **product key during installation**, use the key from the MyAbandonware page comments.

The game will install to:
```
~/.wine/drive_c/Program Files (x86)/EA Games/Harry Potter/
```

---

## Step 5 — Fix the Missing System Files

**This is a critical step that is easy to miss.** The installer does not copy the `System` folder contents from the disc. After installation, the `System` folder exists but is empty of the actual game executable and engine files.

Copy the System folder from the mounted ISO:

```bash
cp -r /mnt/hp1/System ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/
```

Also copy all game asset folders:

```bash
cp -r /mnt/hp1/Maps ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/
cp -r /mnt/hp1/Sounds ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/
cp -r /mnt/hp1/Textures ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/
cp -r /mnt/hp1/Music ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/
```

> **Note:** The actual game executable is `HP.exe` inside the `System` folder — NOT `HarryPotter.exe` (which is just a launcher stub). This tripped us up early on.

---

## Step 6 — Download the DRM-Free Replacement Executable

This is the most important fix. The original `HP.exe` uses SafeDisc copy protection (secdrv.sys), which is completely incompatible with modern Linux/Wine. Without this step, the game will always crash with error `c0000142` referencing `SecDrv`.

The open-source replacement `HP.exe` was created by the Harry Potter Games Archive Project and is hosted on the Internet Archive. It is an improved version that also enables selecting modern render devices (like DirectX 11), which the original locked to DirectX 7 only.

### Download from Internet Archive

Browse and download the HP1 (Philosopher's Stone / Sorcerer's Stone) replacement executable from the community collection on Internet Archive:

**https://archive.org/details/harry-potter-pc-games-no-cd-cracks**

On that page, open the **Files** section and download the `HP.exe` file for Harry Potter 1 (Sorcerer's Stone). It is located in the `SS/` subdirectory. Save it to your Downloads folder as `HP_replacement.exe`.

> **Note for forks of this guide:** Point readers to the Internet Archive collection page above rather than hardcoding direct download URLs to game binaries. Archive paths can change over time.

### How we found the correct file path (useful diagnostic trick)

The Internet Archive organizes files in subdirectories, and paths can change over time. If the file layout is unclear, download the archive's XML file listing from the collection page (look for the `*_files.xml` link under Files, or construct it from the item identifier):

```bash
# Replace ARCHIVE_ITEM_ID with the identifier shown on the collection page
wget "https://archive.org/download/ARCHIVE_ITEM_ID/ARCHIVE_ITEM_ID_files.xml" -O ~/Downloads/files.xml
grep -i "name\|HP.exe" ~/Downloads/files.xml
```

This shows every file with its exact subdirectory path. For HP1, the replacement executable was at `SS/HP.exe` when this guide was written.

Now replace the original copy-protected executable:

```bash
cp ~/Downloads/HP_replacement.exe ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/System/HP.exe
```

---

## Step 7 — Download and Apply Modern Fixes

Go back to the **MyAbandonware page** for the game and scroll down to the **"Game Extras"** section. Download all of these:

- **Fix — Unreal Engine DirectX 11 Renderer** (~860 KB) — enables DirectX 11 rendering on modern hardware
- **Fix — Unofficial Widescreen Patch (for US version)** (~6 MB) — widescreen support
- **Fix — Harry Potter Settings tool** (~938 KB) — GUI tool to change resolution, FOV, widescreen
- **Fix — Mouse Fix and Strafing mod** (~9 MB) — fixes mouse sensitivity and enables strafing

Extract and apply all of them. The fix files have specific subdirectory structures, so copy them carefully:

```bash
cd ~/Downloads

unzip "Harry-Potter-and-the-Sorcerer-s-Stone_Fix_Win_EN_DirectX-11-Renderer.zip" -d dx11
unzip "Harry-Potter-and-the-Sorcerer-s-Stone_Fix_Win_EN_Widescreen-Patch.zip" -d widescreen
unzip "Harry-Potter-and-the-Sorcerer-s-Stone_Fix_Win_EN_Harry-Potter-Settings-tool.zip" -d hpsettings
unzip "Harry-Potter-and-the-Sorcerer-s-Stone_Fix_Win_EN_Mouse-Fix-and-Strafing-mod.zip" -d mousefix
```

The DirectX 11 renderer has a specific folder structure — the HP1-specific DLL is inside `Harry Potter 1/`, while shared files are in `Common/`. Copy them both:

```bash
GAMEDIR=~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/System

cp ~/Downloads/dx11/Common/* "$GAMEDIR/"
cp -r ~/Downloads/dx11/Common/d3d11drv "$GAMEDIR/"
cp ~/Downloads/dx11/Harry\ Potter\ 1/* "$GAMEDIR/"
cp ~/Downloads/widescreen/System/* "$GAMEDIR/"
cp ~/Downloads/hpsettings/* "$GAMEDIR/"
cp ~/Downloads/mousefix/Mouse\ fix\ and\ strafing\ mod/System/* "$GAMEDIR/"
```

> **Note:** When unzipping, you may be prompted to replace existing files. Answer `A` (All) to replace everything.

---

## Step 8 — Copy All Game Assets from the ISO

(You may have done this in Step 5 already — double-check these folders exist and are not empty.)

```bash
ls ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/Maps/
ls ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/Sounds/
ls ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/Textures/
```

If any are empty or missing, re-run the copy commands from Step 5.

---

## Step 9 — Add the Game to Lutris

1. Open **Lutris** (search for it in your applications)
2. Click the **"+"** button → **"Add locally installed game"**
3. Fill in:
   - **Name:** Harry Potter and the Philosopher's Stone
   - **Runner:** Wine
4. Go to the **"Game options"** tab:
   - **Executable:** Browse to `~/.wine/drive_c/Program Files (x86)/EA Games/Harry Potter/System/HP.exe`
   - **Wine prefix:** `~/.wine`
5. Click **Save**

Lutris will automatically fetch the box art. The game should appear in your library.

---

## Step 10 — Configure the Game (Resolution + Language)

The game stores its settings in:
```
~/.wine/drive_c/users/YOUR_USERNAME/Documents/Harry Potter/HP.ini
```

**Do not confuse this with** `~/.wine/drive_c/Program Files (x86)/EA Games/Harry Potter/System/Default.ini` — both files matter.

### Set resolution (windowed mode recommended for ultrawide monitors)

The game was designed for 4:3 aspect ratio. On an ultrawide (21:9) monitor, fullscreen will have black areas or stretch badly. Windowed mode at 1024×768 is the sweet spot:

```bash
HPINI=~/.wine/drive_c/users/$(whoami)/Documents/Harry\ Potter/HP.ini

sed -i 's/StartupFullscreen=True/StartupFullscreen=False/g' "$HPINI"
sed -i 's/UseFullscreen=true/UseFullscreen=false/g' "$HPINI"
sed -i 's/WindowedViewportX=640/WindowedViewportX=1024/g' "$HPINI"
sed -i 's/WindowedViewportY=480/WindowedViewportY=768/g' "$HPINI"
```

### Set language to Spanish (text only — see Spanish Language Notes below)

This requires changes in **two** files. We discovered through trial and error that changing only one of them was not enough:

```bash
HPINI=~/.wine/drive_c/users/$(whoami)/Documents/Harry\ Potter/HP.ini
DEFAULTINI=~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/System/Default.ini

# In the user HP.ini:
sed -i 's/Language=int/Language=spa/g' "$HPINI"
sed -i 's/Language=ESN/Language=spa/g' "$HPINI"  # in case it was set to ESN earlier
sed -i 's/CurLanguageName=ENGLISH/CurLanguageName=SPANISH/g' "$HPINI"

# In the System Default.ini:
sed -i 's/Language=int/Language=spa/g' "$DEFAULTINI"
```

Also copy the Spanish language files from the System folder into the main System folder (they come from the `1` subfolder inside System, which corresponds to Spanish in the US version):

```bash
cp ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/System/1/*.spa \
   ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/System/
```

---

## Step 11 — Launch and Play

Launch from terminal:

```bash
wine ~/.wine/drive_c/Program\ Files\ \(x86\)/EA\ Games/Harry\ Potter/System/HP.exe
```

Or click **Play** in Lutris.

A **Video Configuration** dialog will appear. Select **"Direct3D Support"** and click **Next**. The game will launch.

### To kill Wine processes if the game freezes or gets stuck:

```bash
pkill -f wine
```

---

## Spanish Language Notes

The **US version** of the game includes **Spanish text** but **English audio**. This is a limitation of the specific disc version, not something that can be fixed through configuration.

What works with the Spanish language setting (`Language=spa`):
- All menus and UI text
- Subtitles and dialogue text boxes
- Item names and descriptions
- Credits

What does NOT work (English audio only in this version):
- All voiced dialogue (Harry, Dumbledore, etc.)
- Cutscene narration

To get Spanish voice audio, you would need the **European Spanish release** of the game, which is a separate disc with different audio files. That version is not covered in this guide.

---

## Known Issues and Limitations

**Alt-Tab:** Avoid alt-tabbing out of the game while it's running. This can cause audio to mute and the game to become unstable. If you need to switch windows, save your game first.

**Ultrawide monitors:** The game does not support 21:9. Run it in windowed mode (as configured above) to preserve the 4:3 aspect ratio. Lutris can handle window positioning.

**Fullscreen on ultrawide:** If you try fullscreen, the game renders in a small area in the top-left corner of the screen. This is because the game sets resolution before Wine can properly scale it. Windowed mode avoids this entirely.

**Video config dialog on every launch:** The Video Configuration dialog ("Direct3D Support / S3 MeTaL / Software Rendering") appears every time you launch. Always select Direct3D Support. This is normal behavior.

**Save games:** Save files are stored at:
```
~/.wine/drive_c/Program Files (x86)/EA Games/Harry Potter/save/
```

**Controller support:** The game has limited native controller support. If you need a gamepad, install **AntiMicroX** (`sudo apt install antimicrox`) to map controller buttons to keyboard keys.

---

## Key Findings and Diagnostic Tricks

These are the most valuable things we discovered during this process:

### 1. Finding files on Internet Archive when URLs are broken

When direct download URLs returned 404, we found the correct paths by downloading the archive's XML file listing:

```bash
wget "https://archive.org/download/ARCHIVE_ITEM_ID/ARCHIVE_ITEM_ID_files.xml" -O files.xml
grep "name" files.xml
```

This shows every file in the archive with its exact path, which you can then construct into a proper download URL.

### 2. The language requires TWO ini file changes

Setting `Language=spa` in only the user's `HP.ini` (in Documents) was not enough. The game also reads from `Default.ini` in the System folder. Both need to be updated.

### 3. The language code is `spa`, not `ESN`

We tried `Language=ESN` (which seemed logical for Español) but the game ignored it. The correct Unreal Engine language code for this version turned out to be `spa`.

### 4. The Spanish files are in a numbered subfolder

The US ISO contains language files organized in numbered subfolders:
- `System/0/` — English `.int` files
- `System/1/` — Spanish `.spa` files

These need to be manually copied to the main `System/` directory for the game to find them.

### 5. The installer doesn't copy System files

Despite appearing to install successfully, the original installer leaves the `System/` folder empty of all engine DLLs and the game executable. Everything must be manually copied from the mounted ISO.

### 6. SafeDisc cannot be worked around — it must be replaced

No Wine configuration, compatibility mode, or dll override can make SecDrv/SafeDisc work on Linux. The only solution is replacing `HP.exe` with the open-source DRM-free replacement executable from Internet Archive. There is no other way.

---

## Where to Share and Legal Notes

### Is this legal to share?

This guide itself — the instructions, commands, and findings — is absolutely fine to share anywhere. Knowledge is not copyrightable.

**Do not include:**
- Product keys or license serial numbers in your post
- The ISO or game files themselves
- Direct download links to game binaries (link to the Internet Archive collection page instead)

Point people to MyAbandonware and the Internet Archive for the actual files.

### Where to share

**GitHub (recommended):** A public repository with this markdown file is an excellent way to share it. It's searchable, versioned, and people can open issues or submit pull requests with improvements. A good repo structure would be just a `README.md` with this content.

**MyAbandonware comments:** The comments section of the game's download page is where many people find help. Posting a summary with a link to your GitHub repo is a great approach — it catches people at exactly the moment they need it.

**Other good places:**
- **PCGamingWiki** — You can add a note to the Harry Potter 1 page there linking to your guide
- **Reddit r/linux_gaming** — Active community that helps with Wine/Lutris issues
- **Lutris forums / GitHub issues** — If there's ever an official Lutris installer script for this game, your findings could help improve it
- **WineHQ AppDB** — The database of Wine-compatible applications; you can add a report for this game

### A note for future readers

This guide was written on June 21, 2026. URLs change. Archive.org reorganizes. MyAbandonware may update their file listings. If something doesn't work exactly as described, the **diagnostic tricks in the Key Findings section** (especially the XML listing trick for Archive.org) should help you find the current correct paths. And if you find a better way to do any of this — please share it. That's the whole point.

---

*Written from personal experience. Not affiliated with EA Games, Warner Bros., J.K. Rowling, the Internet Archive, or MyAbandonware. Harry Potter is a trademark of Warner Bros. Entertainment.*
