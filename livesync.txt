# StS2 Live-Sync mirror — copies the game's save into a Chrome-readable folder.
# Keep this window open (minimized is fine) while you play.
$src = Join-Path $env:APPDATA 'SlayTheSpire2'
$dst = Join-Path $env:USERPROFILE 'StS2Sync'
$mirror = Join-Path $dst 'saves\current_run.save'
New-Item -ItemType Directory -Force -Path (Join-Path $dst 'saves\history') | Out-Null

Write-Host ''
Write-Host '  StS2 Live-Sync mirror is RUNNING' -ForegroundColor Green
Write-Host "  Game saves : $src"
Write-Host "  Mirror     : $dst   <- pick THIS folder in the advisor's Live-sync"
Write-Host '  Minimize this window while you play. Press Ctrl+C to stop.'
Write-Host ''

while ($true) {
  try {
    $save = Get-ChildItem -Path $src -Recurse -Filter 'current_run.save' -ErrorAction SilentlyContinue |
            Sort-Object LastWriteTime -Descending | Select-Object -First 1
    if ($save) {
      if (-not (Test-Path $mirror) -or ($save.LastWriteTime -gt (Get-Item $mirror).LastWriteTime)) {
        Copy-Item $save.FullName $mirror -Force -ErrorAction SilentlyContinue
        Write-Host ("  synced  " + (Get-Date -Format 'HH:mm:ss')) -ForegroundColor DarkGray
      }
    } elseif (Test-Path $mirror) {
      Remove-Item $mirror -Force -ErrorAction SilentlyContinue   # run over -> advisor records the result
      Write-Host '  run ended - mirror cleared' -ForegroundColor Yellow
    }
    # mirror run-history files so the advisor can read win/loss results
    Get-ChildItem -Path $src -Recurse -Filter '*.run' -ErrorAction SilentlyContinue | ForEach-Object {
      $target = Join-Path $dst ('saves\history\' + $_.Name)
      if (-not (Test-Path $target)) { Copy-Item $_.FullName $target -Force -ErrorAction SilentlyContinue }
    }
  } catch { }
  Start-Sleep -Seconds 2
}
