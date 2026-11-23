# Making the receiver's fixed action inert

A closed vocabulary is only as narrow as the action behind each word. Before naming a tool as the
fixed action, list every way that tool can run code or reach somewhere else, and turn each off
explicitly on the command line (not by trusting the machine's configuration, which may change).

## git clone (the "set up project" action)
Ways a clone can execute or reach things, and the switch for each:

| Path | Risk | Switch |
| --- | --- | --- |
| Hooks | A machine-wide hooks path (`core.hooksPath`) can run a dispatcher on checkout | `-c core.hooksPath=/dev/null` |
| Submodules | Fetch further URLs chosen by the repository | `--no-recurse-submodules` (and never `submodule update`) |
| Transports | `file://`, `ext::` and others can read local paths or run programs | `-c protocol.allow=never -c protocol.https.allow=always` |
| LFS / smudge filters | Filters named in `.gitattributes` run configured programs on checkout | `GIT_LFS_SKIP_SMUDGE=1`; do not configure filters for this action |
| fsmonitor | Can run a configured program | `-c core.fsmonitor=false` |
| Credential prompts | Hang an unattended process | `GIT_TERMINAL_PROMPT=0` |
| Shell | Argument injection through a name | run the program with an argument list, never through a shell; validate the name first |

The URL is built by the receiver from a fixed owner and a validated repository name, over https.

Credentials: the clone uses the receiving machine's existing git sign-in (credential helper); on
macOS that helper may live in the login Keychain, readable only inside the user's desktop session,
so probe an unattended private clone once before relying on it.

Classify failures from the exit status and the documented stderr phrases: authentication or
"repository not found" are permanent; network, timeout and 5xx are transient.

Clone into a temporary folder that the receiver's own discovery cannot see (e.g. a dot-folder
that is not itself a project, one level below the root the scanner lists; never a sibling that
matches the scanner's predicate), check the result (e.g. the project predicate holds), then
rename into place. Never overwrite or delete an existing destination; an existing destination with
the same origin is "already true", anything else is a refusal.

Afterwards nothing opens, builds, installs or starts anything in the cloned tree.
