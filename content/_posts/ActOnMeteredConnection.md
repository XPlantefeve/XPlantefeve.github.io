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
