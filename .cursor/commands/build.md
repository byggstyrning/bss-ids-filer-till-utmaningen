# Build Revit.IFC.Export

Bygger Revit.IFC.Export projektet i Debug x64 konfiguration.

```powershell
# Hitta MSBuild
$msbuildPath = "${env:ProgramFiles}\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe"
if (-not (Test-Path $msbuildPath)) {
    $msbuildPath = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe"
}

if (-not (Test-Path $msbuildPath)) {
    Write-Host "❌ MSBuild hittades inte. Bygg projektet manuellt i Visual Studio (Ctrl+Shift+B)." -ForegroundColor Red
    exit 1
}

# Bygg projektet
$workspaceRoot = Split-Path -Parent (Split-Path -Parent $PSScriptRoot)
$solutionPath = Join-Path $workspaceRoot "revit-ifc\Revit.IFC.sln"
& $msbuildPath $solutionPath /p:Configuration=Debug /p:Platform=x64 /t:Revit.IFC.Export /m /nologo /v:minimal

if ($LASTEXITCODE -eq 0) {
    $dllPath = Join-Path $workspaceRoot "revit-ifc\Source\Revit.IFC.Export\bin\x64\Debug\Revit.IFC.Export.dll"
    if (Test-Path $dllPath) {
        Write-Host "✅ Build lyckades!" -ForegroundColor Green
    } else {
        Write-Host "⚠️ Build lyckades men DLL hittades inte." -ForegroundColor Yellow
    }
} else {
    Write-Host "❌ Build misslyckades." -ForegroundColor Red
    exit 1
}
```

