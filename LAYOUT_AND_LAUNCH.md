# Where to store the files & how to launch Personal Diary

## 1. Recommended folder layout (put everything in ONE app folder)

Create a single folder for the app — your diary data stays separate in your home folder.

### Windows
```
C:\PersonalDiary\              <-- APP folder (program files live here)
    diary_local.jnlp           (edit codebase -> file:///C:/PersonalDiary/)
    personal-diary.jar         (the built, signed application jar)
    lib\                       (extra jars if any, e.g. pdf-export.jar)

C:\Users\<You>\PersonalDiaryData\   <-- DATA folder (created automatically)
    entries\2026-10-16.txt etc.     (your diary entries + autosave)
```

### Linux / Mac
```
~/PersonalDiary/               <-- APP folder
    diary_local.jnlp           (codebase -> file:///home/<you>/PersonalDiary/)
    personal-diary.jar
    lib\

~/PersonalDiaryData/           <-- DATA folder (auto-created; your entries)
```

Rule of thumb: **app files together in one folder; never edit the .jnlp path without matching where the jar actually is.**

## 2. What each file does
| File | Where it goes | Purpose |
|---|---|---|
| `diary.jnlp` | keep as template | server/web version (example.com placeholder) |
| `diary_local.jnlp` | app folder | launcher you actually double-click |
| `personal-diary.jar` | same folder as the .jnlp | the application itself |
| Your diary entries | `~/PersonalDiaryData` (or `%USERPROFILE%\PersonalDiaryData`) | auto-saved text per day |

## 3. Edit the JNLP before launching
Open `diary_local.jnlp` in Notepad and change this ONE line to match your real folder:

Windows example:
```xml
<jnlp spec="1.0+" codebase="file:///C:/PersonalDiary/" href="diary_local.jnlp">
```
Linux example:
```xml
<jnlp spec="1.0+" codebase="file:///home/yourname/PersonalDiary/" href="diary_local.jnlp">
```
(Trailing slash is required.)

## 4. Launch options (pick one)

**A. OpenWebStart / IcedTea-Web (true JNLP experience)**
1. Install Java 17 (Adoptium Temurin) and OpenWebStart (openwebstart.com).
2. Double-click `diary_local.jnlp` (or right-click → Open with → OpenWebStart / `javaws`).

**B. Simplest — run the jar directly (no JNLP needed)**
```
java -jar C:\PersonalDiary\personal-diary.jar --paper=A4 --ruled-lines=true --autosave=true
```
You can make `launch.bat` (Windows) / `launch.sh` (Linux) with that line and double-click it.

**C. Browser link** — only if you host the folder on a web server; not needed for personal use.

## 5. If it still says "Unable to open application"
1. Check the exact error dialog screenshot/message.
2. Run with verbose log: `javaws -Xverbose diary_local.jnlp` (OpenWebStart) or `itw-javaws -Xverbose ...`.
3. Common causes:
   - jar name/path mismatch with `codebase` → download error
   - unsigned jar + `<all-permissions/>` → security block (sign it or remove all-permissions if app doesn't need file access... but diary needs file access, so sign it)
   - Old Oracle `javaws` removed in JDK 11+ → use OpenWebStart
4. In OpenWebStart settings, add `file:///C:/PersonalDiary/` to deployment.security.allowed.codebases if blocked.
