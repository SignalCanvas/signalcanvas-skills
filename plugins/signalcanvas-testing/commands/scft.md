Run the SignalCanvas label scanner on a specific patch file and report any label propagation gaps found.

The file path to scan is: $ARGUMENTS

Steps:
1. If the argument is a .zip file, extract it to a temp directory first and find the .patch file inside
2. Run: `npm run scan:labels -- <path-to-patch-file>` (also pass the .layout.json sidecar if present alongside the .patch file)
3. Report the results — if issues are found, summarize by device and explain what labels are missing and where they should be coming from
