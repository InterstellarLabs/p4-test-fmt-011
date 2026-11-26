# C++ sample VOBs for migration testing

Three mock VOBs (JSON fixtures in `..\`) emulate `cleartool lshistory` output for
realistic C++ projects. Run them through the converter with the bundled mock
`cleartool`; no ClearCase installation is needed.

| VOB tag | Model | Content |
| --- | --- | --- |
| `/vobs/fmt_sample` | **UCM** | Formatting library. Activities (`%[activity]p`), baselines (`FMT_REL_*`), dev-stream branch `fmt_dev_parser`. |
| `/vobs/cxxopts_sample` | **Base ClearCase** | Option-parser library. VOB labels (`CXX_REL_*`), user-defined attribute types (`CR_ID`, `ReviewStatus`), branch `parser_rework`, a config spec. |
| `/vobs/build_common` | Shared VOB | Build policy referenced from the other two VOBs (see their `config/` files), for cross-VOB dependency discussions. |

## What the data models (ClearCase semantics)

- Elements are created with `mkelem` (`/main/0`); first check-in is `/main/1`.
- Branches are created with `mkbranch`; branch versions look like `/main/<branch>/N`.
- Labels are VOB-wide types applied to the version of **every** element in the
  labeled configuration, including unchanged ones, so one label spans many
  check-ins.
- Logins (`jdoe`, `rpatel`, ...) are fictional ClearCase users mapped to names
  and emails in `users.csv`; comments are ordinary check-in comments, with
  ClearQuest-style IDs (`FMT00000114`, `CXX00000212`).
- A text file removed with `rmname`, binary elements, and a shared-VOB
  dependency are included as edge cases.

## Ignoring files (ClearCase vs Git)

ClearCase has no `.gitignore`. Files never added with `mkelem` stay **view-private**,
and a pre-op trigger can refuse specific names. The fixtures model this:

- Each VOB carries a trigger type, `NO_BUILD_ARTIFACTS` (pre-op `mkelem`, all
  elements), listed by the mock's `lstype -kind trtype -invob VOB -short`.
- `/vobs/build_common` holds the trigger script (`triggers/block_artifacts.sh`)
  and its deny-list (`triggers/mkelem_denylist.txt`, two versions).
- Each project VOB also holds **legacy artifacts checked in before the trigger
  existed** (`*.obj`/`*.exe`, `*.keep`/`*.contrib` merge leftovers, `*.log`).
  They are real elements with labels, so a naive conversion carries them into Git.

`cc2gh.py` can leave them out and mirror the rule for Git:

```powershell
py .\cc2gh.py --mock --vob-root \vobs\fmt_sample --out .\out `
  --ignore-from-clearcase /vobs/build_common/triggers/mkelem_denylist.txt
```

- Matching versions are skipped and reported; they are recorded under `ignored`
  in the `clearcase-metadata` sidecar, along with the VOB's trigger names.
- A `.gitignore` with the same patterns is committed on the tip of `main` only,
  so label tags and branches remain exact ClearCase snapshots. Use `--no-gitignore`
  to skip it, or `--ignore PATTERN` for extra patterns.
- Without these flags the artifacts are converted like any other element.

## Known simplifications (not real ClearCase behavior)

- Real ClearCase also versions **directories** and records `rmname` and renames
  on the parent directory version; these fixtures record file elements only.
  Renames are not modeled: a renamed element keeps its history, which a
  file-level `lshistory -recurse` shows under its current name.
- UCM is represented only through `%[activity]p` and baseline-named labels; the
  mock does not implement streams, projects, or baseline dependency resolution.
- Source content comes from MIT-licensed upstream projects ({fmt}, cxxopts).
  Upstream URLs, PR numbers, and Git/GitHub metadata files were removed or
  neutralized. Histories, authors, dates, and comments are synthetic.
- Binary elements are small test blobs; nothing here performs Git LFS migration.

## Run

```powershell
foreach ($v in 'build_common','fmt_sample','cxxopts_sample') {
  py .\cc2gh.py --mock --vob-root "\vobs\$v" --out .\out
}
```

Use a new `--out` directory if a target repository already exists. Add
`--ignore-from-clearcase /vobs/build_common/triggers/mkelem_denylist.txt` to
exclude the legacy build artifacts (see above).

## Reference snapshots

`fmt-12.2.1` and `cxxopts-v3.3.1` are the original MIT-licensed source
snapshots (with upstream `LICENSE` files), kept for reference only; Git and
GitHub metadata files (`.github`, `.gitignore`, `.travis.yml`) were removed.
Nothing reads them: the fixtures embed their own file contents.


## Test commit

This section contains a sample commit used to verify the conversion process from Perforce to Git. It does not represent any real development history and is included solely for testing purposes.