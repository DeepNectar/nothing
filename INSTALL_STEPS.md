# Personal Diary (JNLP) — Why it failed & step-by-step install

## Why you got "Unable to open the application"

| # | Cause | Explanation |
|---|-------|-------------|
| 1 | **Fake codebase URL** | The JNLP pointed to `https://example.com/diary/` — that server doesn't exist, so the launcher could not download anything. |
| 2 | **JAR does not exist** | `personal-diary.jar` was never built/placed next to the JNLP. A JNLP is only a *launcher* — the actual program (jar) must exist. |
| 3 | **Java Web Start removed** | Oracle removed Java Web Start (`javaws`) from JDK 11+. On modern systems `.jnlp` files have no handler, hence "unable to open". Use **IcedTea-Web** instead. |
| 4 | **Unsigned JAR + all-permissions** | If `<all-permissions/>` is requested, the JAR **must be signed**, otherwise launch is blocked by security. |
| 5 | **Wrong main-class** | `com.diarysoft.personaldiary.DiaryApp` must exactly match a real class with `public static void main(String[] args)` inside your jar. |

## Step-by-step installation (Windows/Linux/macOS)

### Step 1 — Install Java 8 or newer
```
java -version   # if this fails, install first
```
- Easiest: install **Adoptium Temurin 17** → https://adoptium.net (check "Add to PATH" / "Set JAVA_HOME" on Windows).

### Step 2 — Install IcedTea-Web (the .jnlp opener)
- **Windows:** download the MSI installer from https://openwebstart.com/download/ and run it. After that, double-clicking a `.jnlp` file works.
- **Linux (Debian/Ubuntu):** `sudo apt install icedtea-netx`
- **macOS:** use OpenWebStart from https://openwebstart.com

### Step 3 — Put the real files in one folder
Create folder `PersonalDiary` (e.g. `C:\PersonalDiary` or `~/PersonalDiary`) and put inside:
- `diary_local.jnlp`  (the fixed local launcher in this repo)
- `personal-diary.jar` (**your built application jar** — see Step 4; without a real jar nothing can launch)

Edit `diary_local.jnlp` line:
```xml
<jnlp spec="1.0+" codebase="file:///C:/PersonalDiary/" href="diary_local.jnlp">
```
(use forward slashes; Mac example: `file:///Users/YOU/PersonalDiary/`)

### Step 4 — Build the jar (if you don't have one yet)
```bash
mkdir -p src/com/diarysoft/personaldiary
# put DiaryApp.java there (must contain public static void main)
javac -d out src/com/diarysoft/personaldiary/*.java
jar cfe personal-diary.jar com.diarysoft.personaldiary.DiaryApp -C out .
mv personal-diary.jar ~/PersonalDiary/     # into the folder from Step 3
```

### Step 5 — Sign the jar (required because of `<all-permissions/>`)
```bash
keytool -genkeypair -alias diary -keyalg RSA -keysize 2048 \
        -validity 3650 -keystore diary.keystore -storepass changeit
jarsigner -keystore diary.keystore -storepass changeit \
        personal-diary.jar diary
```
(Alternative for quick testing: remove `<security><all-permissions/></security>` from the JNLP — app then runs sandboxed.)

### Step 6 — Launch
- Double-click `diary_local.jnlp`, **or**
- Terminal: `javaws diary_local.jnlp` (IcedTea-Web), or
- Direct test without JNLP at all: `java -jar personal-diary.jar --paper=A4`

### Step 7 — If it still fails, check these
1. Run from a terminal to see the real error: `javaws -Xverbose diary_local.jnlp`
2. Clear the cache: `javaws -uninstall` … or delete `~/.config/icedtea-web/cache/`
3. Confirm paths: the jar name in JNLP must equal the file name exactly.
4. Confirm Java version matches `<j2se version="1.8+">`.
5. Windows firewall / proxy blocking `file://`? Not applicable — but browser downloads of `.jnlp` may need the MIME type `application/x-java-jnlp-file` if served over HTTP later.

## Simplest alternative
If you only want to run your own diary app locally, skip JNLP entirely:
```bash
java -jar personal-diary.jar --paper=A4
```
and make a desktop shortcut to that command. JNLP is mainly useful for distributing apps over a web server.
