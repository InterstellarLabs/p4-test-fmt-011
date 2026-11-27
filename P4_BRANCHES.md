# Perforce Branch Commands

List branch specifications:

```powershell
p4 branches
```

List streams:

```powershell
p4 streams
```

Create and populate the streams:

```powershell
$streamSpec = @'
Stream:	//streamsDepot/feature_1.2.3
Owner:	vpulagam
Name:	feature_1.2.3
Parent:	//streamsDepot/mainline
Type:	development
Description:
	Feature 1.2.3 stream.
Options:	allsubmit unlocked toparent fromparent mergedown
ParentView:	inherit
Paths:
	share ...
'@
$streamSpec | p4 stream -i
p4 populate -d "Populate feature_1.2.3 stream from mainline" -r -S //streamsDepot/feature_1.2.3

$streamSpec = @'
Stream:	//streamsDepot/Release-1.2
Owner:	vpulagam
Name:	Release-1.2
Parent:	//streamsDepot/mainline
Type:	release
Description:
	Release 1.2 stream.
Options:	allsubmit unlocked toparent fromparent mergedown
ParentView:	inherit
Paths:
	share ...
'@
$streamSpec | p4 stream -i
p4 populate -d "Populate Release-1.2 stream from mainline" -r -S //streamsDepot/Release-1.2
```

Switch and sync one stream at a time:

```powershell
p4 switch //streamsDepot/Main
p4 sync

p4 switch //streamsDepot/feature_1.2.3
p4 sync

p4 switch //streamsDepot/Release-1.2
p4 sync
```

Set and sync the Main branch:

```powershell
p4 switch //streamsDepot/Main
p4 sync
```

For VS Code Source Control:

```text
Set "perforce.activationMode" to "autodetect" in .vscode/settings.json,
then run "Developer: Reload Window" from the Command Palette.
```
