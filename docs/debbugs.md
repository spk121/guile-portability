# Filing a Bug and Sending a Patch with Emacs, Debbugs, mu4e, and Git

This guide assumes you're using Fedora with Emacs, `mu4e` for email
(with `mbsync` fetching mail to a local Maildir), and `git` for version
control.  This guide focuses on GNU Guile as the example project, but
the process applies to any Debbugs-tracked package.

# Using the Debbugs Emacs Interface to View GNU Guile Bugs

The GNU Guile project tracks its bugs using the Debbugs system at
`debbugs.gnu.org`, and you can explore these bugs directly in Emacs using
the `debbugs.el` package. This guide walks you through installing,
configuring, and using `debbugs.el` to list bugs for the "guile" package,
leveraging your Fedora environment.

## Step 1: Install the Debbugs Package

`debbugs.el` is an Emacs package for interacting with the GNU Debbugs
bug tracker. It is available via GNU ELPA, Emacs’ default package
repository, and requires no additional Fedora packages beyond your
existing Emacs installation.

1. **Open Emacs Package Manager**:
   - Run `M-x package-list-packages` (press `Alt+x`, type
     `package-list-packages`, and press `RET`).
   - If ELPA isn't configured, add this to your `~/.emacs` or
     `~/emacs.d/init.el`:

     ```emacs-lisp
     (require 'package)
     (add-to-list 'package-archives '("gnu" . "https://elpa.gnu.org/packages/"))
     (package-initialize)
     ```

     Then reload Emacs (`M-x eval-buffer` or restart) to apply this.

2. **Install `debbugs`**:
   - In the package list, search for `debbugs` with `C-s debbugs`
   - Mark it for installation with `i`, then execute with `x`.
   - Alternatively, install directly: `M-x package-install RET debbugs RET`.
   - After installation, Emacs downloads and compiles `debbugs.el`

- *Note*: Fedora’s Emacs (e.g., `emacs-29.4-5.fc41`) supports ELPA out of the box, so no additional dependencies are needed.

## Step 2: Basic Configuration

To use `debbugs.el`, load it in your Emacs config:
I added a minimal setup to your Emacs config (`~/.emacs`) to load
`debbugs.el`:

1. Edit your Emacs config, adding the following to `~/.emacs` or `~/.emacs.d/init.el`

   ```emacs-lisp
   (require 'debbugs)
   ```

## Step 3: View Guile Bugs

Debbugs organizes bugs by "packages," and GNU Guile’s bugs are tracked
under the `guile` package. Use the `debbugs-gnu` command to fetch and
display them.

1. **Run the Command**:
   - Type `M-x debbugs-gnu RET`.
   - By default, it shows bugs for the "emacs" package, but you can
     specify "guile" instead.

2. **Specify the Guile Package**:
   - With a prefix argument (`C-u M-x debbugs-gnu RET`), you’re
     prompted for:
     - **Severities**: Press `RET` for all (or type `minor`, `normal`, etc., separated by commas).
     - **Packages**: Type `guile RET` to filter for Guile bugs.
     - **Archivedp**: Type `nil RET` for active bugs only (or `t` for archived).
     - **Suppress**: Press `RET` (or list tags to suppress, e.g. `patch`).
     - **Tags**: Press `RET` for no tag filter (or enter tags like `patch`, `confirmed`).

   - Example input: `RET guile RET nil RET RET RET`.
   - This fetches all active Guile bugs

3. **View Results**:
   - Emacs fetches bugs via the Debbugs SOAP interface and displays
     them in a buffer: `*Debbugs*` or `*Guile Bugs*`
   - The buffer uses the tabulated list mode to display
     them in a tabulated list (e.g., bug number, severity, title).
   - Example output might include bugs like `#54321: guile:
     segfault in module X`.

- The SOAP query can take a moment, especially for many bugs. If Emacs
  is compiled with threading (Fedora’s default since Emacs 27),
  retrieval runs in the background.

## Step 4: Navigate and Explore Bugs

In the `*Debbugs*` buffer:

- **View Details**: Press `RET` on a bug to view its full details (emails, status, etc.).
- **Navigate**: Use `n` (next) and `p` (previous) to move between bugs, or `q` to 
  close the buffer.
- **Refresh**: Type `g` to refresh the list.
- **Filter**: Type `f` to narrow by tags or status interactively.
- **Detail**: The details buffer show the bug's email thread, which
  you can reply to directly or view in `mu4e` if synced.

## Step 5: Optional Customization

To default to Guile bugs without `C-u`:

1. Edit you Emacs config:
  * Add this to `~/.emacs` or `~/.emacs.d/init.el`:

   ```emacs-lisp
   (setq debbugs-gnu-default-packages '("guile"))
   ```
  * Reload Emacs

2. **Usage**: Now, `M-x debbugs-gnu RET` shows Guile bugs directly.
  * **Note**: This overrides the default "emacs" package.  For multiple
    packages, you'd change the setting of `debbugs-gnu-default-packages`
    to a list, e.g. `'("guile" "emacs")`.
  
**Note:** This overrides the default "emacs" package focus.

## Step 6: Filing a Bug

To file a new bug for Guile:

1. Compose in mu4e:
  * Open `mu4e` (`M-x mu4e RET`) and press `m` to compose a new message.
  * **To**: `submit@debbugs.gnu.org` which is the general submisson address.
  * **Subject**: A short description (e.g., `guile: it is broken`)
  * **Body**: Describe the issue, including steps to reproduce, Guile
    version (e.g., `guile --version`) and logs
  * Add a pseudo-header at the top of the body

     ```
     Package: guile
     Severity: normal
     ```
     Where the allowed severities are `minor`, `normal`, `serious`,
     `important`, and `grave`.

2. Wait for a confirmation e-mail.
  * Debbugs will assign a bug number (e.g. 54321) and email you back.

### Step 7: Creating and Sending a Patch

To contribute a fix to GNU Guile (or any Debbugs-tracked project), use 
`git` to create a patch and `git send-email` to submit it to the bug 
tracker. This assumes you’ve identified a bug (e.g., `#54321`) from Step 3.

1. **Clone the Guile Repository**:
   - Open a terminal and run:
     ```bash
     git clone https://git.savannah.gnu.org/git/guile.git
     cd guile
     ```
   - This downloads the Guile source code to a `guile` directory.

2. **Make Changes**:
   - Edit the relevant files in your local clone to fix the bug
     (e.g., modify source code in `module/`).
   - Stage and commit your changes:
     ```bash
     git add <file>
     git commit -m "Fix segfault in module X (bug#54321)"
     ```
   - Repeat for multiple commits if your fix spans several logical changes (e.g., code fix, tests).

3. **Install `git send-email`**:
   - Install the `git-email` package on Fedora:
     ```bash
     sudo dnf install git-email
     ```

4. **Configure Git for Email**:
   - Set up your SMTP server (example for Yahoo; adjust for your provider, e.g., Gmail):
     ```bash
     git config --global sendemail.smtpserver smtp.mail.yahoo.com
     git config --global sendemail.smtpencryption tls
     git config --global sendemail.smtpuser spk121@yahoo.com
     git config --global sendemail.smtpserverport 587
     ```
   - Set your identity:
     ```bash
     git config --global user.name "Mike Gran"
     git config --global user.email "spk121@yahoo.com"
     ```
   - **Note**: You can test with `git send-email --smtp-debug=1` if unsure.

5. **Generate Patches**:
   - Create patch files for the commits.
     * If, for example, you want the last 2 commits, you can use `-2`
       for the range:

        ```bash
        git format-patch -2 --cover-letter
        ```
     * Otherwise, you'll have to identify the range using IDs.
       The first ID will be last commit *before* your changes.
       The second ID will be the last commit *of* your changes.

       ```
       git format-patch a1b2c3...d4e5f6 --cover-letter

   - This generates files like `0000-cover-letter.patch`,
     `0001-....patch`, and `0002-....patch` in the current directory.

6. **Edit the Cover Letter**:
   - Open the cover letter in Emacs:
     ```bash
     emacs 0000-cover-letter.patch
     ```
   - Edit the `Subject:` line to include the bug number:
     ```
     Subject: [PATCH 0/2] bug#54321: Fix segfault in module X
     ```
   - Below `*** BLURB HERE ***`, add a description:
     ```
     This patch series fixes a segfault in Guile’s module X by:
     - Patch 1: Adjusts buffer handling to prevent overflow.
     - Patch 2: Adds a test case to verify the fix.
     ```
   - Set the `To:` field to the bug’s email address:
     ```
     To: 54321@debbugs.gnu.org
     ```
   - Save (`C-x C-s`).

7. **Edit Patch Subjects**:
   - Open each patch file (e.g., `0001-....patch`) in Emacs:
     ```bash
     emacs 0001-*.patch
     ```
   - Prepend the bug number to the `Subject:`:
     ```
     Subject: [PATCH 1/2] bug#54321: Adjust buffer handling
     ```
   - Repeat for all patches (e.g., `0002-....patch` becomes
     `[PATCH 2/2] bug#54321: Add test case`).
   - Save each file.

8. **Send the Patches**:
   - From the `guile` directory, send all patches:
     ```bash
     git send-email --to=54321@debbugs.gnu.org --sendmail-cmd=/usr/bin/msmtp *.patch
     ```
   - Enter your SMTP password if prompted (or add
     `git config --global sendemail.smtppass "yourpassword"`
     to avoid this, though less secure).
   - The patches are emailed to `54321@debbugs.gnu.org`, linking them
     to bug `#54321`.
   - **Note**: Use `--dry-run` to test without sending:
     `git send-email --dry-run *.patch`.

9. **Verify in mu4e**:
   - After your mail inbox has updated, open `mu4e`
     ```emacs-lisp
     M-x mu4e RET
     ```
   - Search for your patch submission:
     ```
     bug#54321
     ```
     or
     ```
     from:spk121
     ```
   - You’ll see your sent patches and any replies from maintainers
     (e.g., “Patch applied” or review comments).

- **Additional Tips**:
  - If `git send-email` fails (e.g., SMTP auth issues), check logs
    with `--smtp-debug=1` or send manually via `mu4e`:
    - Compose a new message (`m` in `mu4e`), attach the `.patch`
      files (`C-c C-a`), set `To: 54321@debbugs.gnu.org`, and
      send (`C-c C-c`).
  - Ensure your commits follow Guile’s contribution guidelines
    (e.g., `git log` for style).
