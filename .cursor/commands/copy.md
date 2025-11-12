# Copy DLL to Revit Add-in Folders

Kopierar den byggda DLL:en till Revit 2025 och 2026 add-in mappar.

```powershell
$workspaceRoot = Split-Path -Parent (Split-Path -Parent $PSScriptRoot)
$dllPath = Join-Path $workspaceRoot "revit-ifc\Source\Revit.IFC.Export\bin\x64\Debug\Revit.IFC.Export.dll"

if (-not (Test-Path $dllPath)) {
    Write-Host "❌ DLL hittades inte: $dllPath" -ForegroundColor Red
    Write-Host "   Bygg projektet först med build-kommandot." -ForegroundColor Yellow
    exit 1
}

$fileInfo = Get-Item $dllPath
Write-Host "📦 Kopierar DLL till Revit add-in mappar..." -ForegroundColor Cyan
Write-Host "   Källa: $dllPath" -ForegroundColor Gray
Write-Host "   Senast ändrad: $($fileInfo.LastWriteTime)" -ForegroundColor Gray

$target2025 = "C:\ProgramData\Autodesk\ApplicationPlugins\IFC 2025.bundle\Contents\2025\Revit.IFC.Export.dll"
$target2026 = "C:\ProgramData\Autodesk\ApplicationPlugins\IFC 2026.bundle\Contents\2026\Revit.IFC.Export.dll"

# Kontrollera om Revit är öppet
$revitProcesses = Get-Process -Name "Revit" -ErrorAction SilentlyContinue
if ($revitProcesses) {
    Write-Host "`n⚠️ VARNING: Revit är öppen!" -ForegroundColor Yellow
    Write-Host "   Stäng Revit innan du kopierar DLL:en för att undvika fil-låsningar." -ForegroundColor Yellow
    $continue = Read-Host "Fortsätt ändå? (j/n)"
    if ($continue -ne "j" -and $continue -ne "J") {
        Write-Host "Avbruten." -ForegroundColor Gray
        exit 0
    }
}

# Kopiera till Revit 2025
try {
    if (-not (Test-Path (Split-Path $target2025 -Parent))) {
        New-Item -ItemType Directory -Path (Split-Path $target2025 -Parent) -Force | Out-Null
    }
    Copy-Item -Path $dllPath -Destination $target2025 -Force -ErrorAction Stop
    Write-Host "✅ Kopierad till Revit 2025" -ForegroundColor Green
} catch {
    Write-Host "⚠️ Kunde inte kopiera till Revit 2025: $_" -ForegroundColor Yellow
}

# Kopiera till Revit 2026
try {
    if (-not (Test-Path (Split-Path $target2026 -Parent))) {
        New-Item -ItemType Directory -Path (Split-Path $target2026 -Parent) -Force | Out-Null
    }
    Copy-Item -Path $dllPath -Destination $target2026 -Force -ErrorAction Stop
    Write-Host "✅ Kopierad till Revit 2026" -ForegroundColor Green
} catch {
    Write-Host "⚠️ Kunde inte kopiera till Revit 2026: $_" -ForegroundColor Yellow
}

Write-Host "`n⚠️ VIKTIGT: Starta om Revit för att ladda den nya DLL:en!" -ForegroundColor Yellow
```

