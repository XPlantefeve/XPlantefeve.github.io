---
layout: post
title: Stop Dropbox from using my metered connection
date:   2019-02-03
# excerpt:
log: true
tag:
comments: true
---

```powershell
[void][Windows.Networking.Connectivity.NetworkInformation, Windows, ContentType = WindowsRuntime]
$ConnectionProfile = [Windows.Networking.Connectivity.NetworkInformation]::GetInternetConnectionProfile()

If (
    $ConnectionProfile -and
    ( $cost = $ConnectionProfile.GetConnectionCost() ) -and
    (
        $cost.Roaming -or
        $cost.OverDataLimit -or
        $cost.NetworkCostType -in 'Fixed', 'Variable'
    )
) {
    ps Dropbox | kill
    service CrashPlanService | Stop-Service
}
```