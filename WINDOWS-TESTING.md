# Testing Diwan on Windows

Diwan has not been fully tested on a real Windows machine. Everything Windows-specific in it (finding Claude Code,
folder paths, starting and stopping `claude`, the local event bus, the terminal sessions hooks) was written and tested
on Linux only. This checklist is the first real check. Nothing counts as "works on Windows" until it comes back complete.

**Machine:** Windows 10 (1809 or later) or 11, x64. Ideally a clean machine or VM without Node.js and without
Claude Code. Git for Windows is optional (one run without it is useful).

**Each problem you find becomes its own issue.** Copy the report template at the end, fill it in, and attach the
screenshots. Do not paste your email address or account details from any command output.

## Before you start, send

- Your Windows version (run `winver`).
- Whether Git for Windows is installed.
- Whether Node.js is installed (`where.exe node` in PowerShell).

## Checklist

| # | Do | You should see | Send |
|---|---|---|---|
| W1 | Install `Diwan_0.4.0_x64-setup.exe` | SmartScreen: **More info** → **Run anyway**. After the Welcome page the license page shows with **I Agree** (**Cancel** stops the install). It installs without admin. If it asks for WebView2 it installs it by itself | A screenshot of any message and of the license page |
| W2 | Open Diwan from the Start menu **before** installing Claude Code | The hall is drawn (floor, walls and both tables complete) and a card "Claude Code is not installed" with an `irm ...` command | A screenshot of the whole screen |
| W3 | In PowerShell: `irm https://claude.ai/install.ps1 \| iex`, then `Test-Path "$env:USERPROFILE\.local\bin\claude.exe"` and `where.exe claude` (in the same, old window). Go back to Diwan → **Check again** | `True`, and `where.exe` may not find it. The card now says "Claude Code is not signed in" | The output of both commands + a screenshot |
| W4 | `& "$env:USERPROFILE\.local\bin\claude.exe"` → sign in → quit. Also `& "$env:USERPROFILE\.local\bin\claude.exe" auth status --json \| Select-String loggedIn` and `& "$env:USERPROFILE\.local\bin\claude.exe" --version`. **Check again** | The card goes away | The output (without your email) |
| W5 | ``mkdir "$env:USERPROFILE\my demo"; "apple`nbanana" \| Out-File -Encoding utf8 "$env:USERPROFILE\my demo\notes.txt"``. In Diwan, open a session with `C:\Users\<you>\my demo` | A scribe walks in, and the table is named `my demo` | A screenshot |
| W6 | Also try `~\my demo` and `C:/Users/<you>/my demo/` | The same table (not a second one) | A screenshot |
| W7 | Steps 5 to 9 below (Read, WebFetch + Allow, Bash + Deny, Stop, closing) | The same behaviour as on Linux, and no black console window appears | A screenshot of each step, and the text of any error message |
| W8 | A reply in Arabic: `Reply in Arabic: what is in notes.txt?` | The Arabic text is intact in the reply | A screenshot |
| W9 | Task Manager → Details: after closing the session, and after closing Diwan | No extra `claude.exe` or `cmd.exe` left by Diwan | A screenshot |
| W10 | (Optional; tests the event bus and the scene without Claude) With Diwan open, run the PowerShell script below | Each call answers `ok` = `True`; a scribe walks in and goes to the archive, and after the second event it leaves | The output + a screenshot |
| W11 | (If npm is installed) `npm install -g @anthropic-ai/claude-code`, move the native one aside (`Rename-Item "$env:USERPROFILE\.local\bin\claude.exe" claude.exe.bak`), close and reopen Diwan, open a session | Diwan finds `%APPDATA%\npm\claude.cmd` and works, and closing leaves no orphan `claude.exe` | A screenshot + Task Manager. Rename it back afterwards |
| W12 | (Only if the PC has a second drive, for example `D:`) Make a folder there with a space in its name, for example `D:\my demo 2`, and open a session with it. Also try it as `D:/my demo 2/` | A scribe walks in and the table is named `my demo 2`. The same path in both spellings gives the same table | A screenshot. Say which drive letters the PC has |
| W13 | Open the sessions menu (the **Sessions** button), then look at the model list above **Open** and at the last line of the menu | The list offers Default (Claude Code), Fable 5.1, Opus 5.5, Sonnet 5.5 and Haiku 5.5. The last line reads `0.4.0 · <commit>` (not `DEV`) | A screenshot of the open menu |
| W14 | Pick **Sonnet 5.5**, open a session on `C:\Users\<you>\my demo`, send a short message, then close and reopen Diwan and open the menu again | The session row shows a `Sonnet 5.5` tag, and the list still says Sonnet 5.5 after the restart. If your account has no access to that model, say so and what error you saw | A screenshot before and after the restart |
| W15 | (Tests the local-port fix) With Diwan open, in PowerShell: `Invoke-WebRequest -Method Post -Uri http://127.0.0.1:4317/event -ContentType text/plain -Body x -SkipHttpErrorCheck` (PowerShell 7), or in Windows PowerShell 5.1 the same inside try/catch and print the status code | Status 415 (`expected application/json`), and no scribe appears. W10 with `application/json` still gives `ok` = `True` | The output |
| W16 | Open the install folder: type `%LOCALAPPDATA%\Diwan` in File Explorer's address bar. If it is not there, right-click Diwan in the Start menu → **More** → **Open file location**, then right-click the shortcut → **Open file location** | `LICENSE.txt` and `THIRD-PARTY-NOTICES.txt` are there, and the notices mention `Microsoft Edge WebView2 SDK` and `NSIS 3.13` | A screenshot of the folder |
| W17 | In the sessions menu: **＋ New session in folder…** → **Browse…**, pick `my demo`, press **Open**. Then open **Browse…** again and cancel it | The Windows folder window opens, `C:\Users\<you>\my demo` is written in the field and the session opens at the `my demo` table. Cancelling keeps what was in the field | A screenshot of the folder window and of the field |
| W18 | Press **Show terminal sessions** at the bottom of the sessions menu. Then in PowerShell: `cd "$env:USERPROFILE\my demo"`, run `claude` and send `Read notes.txt`. Then close Diwan with **×**, send `claude` one more message, and run `Select-String -Path "$env:USERPROFILE\.claude\settings.json" -Pattern '4317/hook'`. Open Diwan again and run the same `Select-String` | The button turns into **Stop showing them**. A scribe walks in at the `my demo` table while `claude` reads. After closing Diwan, `claude` shows no `Stop hook error` and `Select-String` prints nothing. After reopening it prints 7 lines | A screenshot of the scribe and the output of both `Select-String` runs |
| W19 | With Diwan open, open Diwan a second time from the Start menu | The second window says that port 4317 is already in use, and the first one keeps working | A screenshot of the message |

### Steps 5 to 9 (for W7)

Send these in the `my demo` session:

| Step | Do | You should see |
|---|---|---|
| 5 | Send `Read notes.txt and tell me its first line.` | The scribe goes to the archive (reading), comes back, delivers at your table (done) and returns to its chair; the reply appears in a card titled **REPLY** |
| 6 | Send `Fetch https://example.com and tell me the page title.` | An approval bubble over the scribe, who waits at your table (blocked). **Allow** → the scribe goes to the window (web), then delivers |
| 7 | Send ``Run `echo hi > hi.txt` `` → **Deny** | Claude replies that it was denied, and the scribe does not stay standing in the queue |
| 8 | Send `Count from 1 to 300, one number per line.` → press **Stop** while it writes | The scribe goes back to idle, and no error message |
| 9 | Close the session (**End session**) | The scribe walks to the entrance and disappears |

### W10 script

```powershell
# W10: posts two AgentEvents to the app's local bus, like a hook would.
$sid = [guid]::NewGuid().ToString()
function Post($type, $tool) {
  $e = @{ id = [guid]::NewGuid().ToString(); ts = (Get-Date).ToUniversalTime().ToString('o');
          agent_id = 'win-test'; agent_kind = 'claude-code'; session_id = $sid; type = $type; project = 'win-test' }
  if ($tool) { $e.tool = $tool }
  Invoke-RestMethod -Method Post -Uri 'http://127.0.0.1:4317/event' -ContentType 'application/json' -Body ($e | ConvertTo-Json)
}
Post 'tool_started' 'Read'
Start-Sleep -Seconds 8
Post 'session_ended' $null
```

## Report template

Copy this into an issue, one issue per problem found (and one issue with the full run).

```markdown
### Diwan Windows test: 0.4.0

- Windows version (winver):
- Git for Windows installed: yes / no
- Node.js installed (where.exe node):
- Claude Code version (claude.exe --version):
- Installer file and its SHA-256 (Get-FileHash):

| # | Result (pass / fail / skipped) | What you saw, if different | Screenshot / output |
|---|---|---|---|
| W1 | | | |
| W2 | | | |
| W3 | | | |
| W4 | | | |
| W5 | | | |
| W6 | | | |
| W7 step 5 | | | |
| W7 step 6 | | | |
| W7 step 7 | | | |
| W7 step 8 | | | |
| W7 step 9 | | | |
| W8 | | | |
| W9 | | | |
| W10 | | | |
| W11 | | | |
| W12 | | | |
| W13 | | | |
| W14 | | | |
| W15 | | | |
| W16 | | | |
| W17 | | | |
| W18 | | | |
| W19 | | | |

Error messages (exact text):

Anything else:
```
