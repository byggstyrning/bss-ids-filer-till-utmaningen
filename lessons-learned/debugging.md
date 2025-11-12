# Guide för Debugging av Custom IFC Exporter

## Problem identifierat

Revit laddar DLL:en från `C:\Program Files\Autodesk\Revit 2026\Revit.IFC.Export.dll` (den installerade versionen) i stället för din custom version från add-in mappen.

## Lösning: Aktivera Symbol Loading och Debugging

### Steg 1: Konfigurera Visual Studio för Symbol Loading

1. **Tools → Options → Debugging → General**
   - ✅ **Avmarkera** "Enable Just My Code"
   - ✅ **Markera** "Enable .NET Framework source stepping"
   - ✅ **Markera** "Enable source server support"

2. **Tools → Options → Debugging → Symbols**
   - Lägg till symbol path:
     ```
     C:\Users\JonatanJacobsson\Downloads\bss-ids-filer-till-utmaningen\revit-ifc\Source\Revit.IFC.Export\bin\x64\Debug
     ```
   - ✅ **Markera** "Load all modules, unless excluded"
   - ✅ **Avmarkera** "Only load specified modules"

### Steg 2: Sätt Breakpoints

1. Öppna filen där du vill debugga (t.ex. `IFCInstanceExporter.cs` rad 2122)
2. Sätt en breakpoint på raden där `PredefinedType` sätts
3. Tryck F5 för att starta debugging

### Steg 3: För att se din custom DLL laddas

**Alternativ A: Kopiera Debug DLL till Revit-mappen (kräver admin)**

Kör PowerShell som Administrator:
```powershell
$source = "C:\Users\JonatanJacobsson\Downloads\bss-ids-filer-till-utmaningen\revit-ifc\Source\Revit.IFC.Export\bin\x64\Debug\Revit.IFC.Export.dll"
$target = "C:\Program Files\Autodesk\Revit 2026\Revit.IFC.Export.dll"
Copy-Item $source $target -Force
```

**Alternativ B: Använd Add-In Bundle (rekommenderat)**

Din custom DLL kopieras automatiskt till add-in mappen efter build. Men Revit kan fortfarande ladda den installerade versionen först.

### Steg 4: Verifiera att din custom DLL laddas

När du startar Revit från Visual Studio, kolla Output-fönstret:
- Du bör se: `Loaded 'C:\Program Files\Autodesk\Revit 2026\Revit.IFC.Export.dll'`
- Om din custom version laddas kommer den att ha din custom timestamp

## För att debugga export-felet

1. **Sätt breakpoints i:**
   - `IFCInstanceExporter.CreateBridgePart` (rad 2122)
   - `Exporter.CreateFacilityPart` (rad 1567-1573)

2. **Starta Revit från Visual Studio (F5)**

3. **Öppna ditt projekt i Revit**

4. **Starta IFC Export**

5. **När breakpoint träffas:**
   - Kontrollera värdet på `predefinedType`
   - Stega igenom koden för att se var felet uppstår

## Om du fortfarande inte ser debug-output

1. **Kontrollera att Debug-konfigurationen är vald:**
   - I Visual Studio: Debug → Configuration Manager
   - Se till att "Debug" är valt för projektet

2. **Kontrollera Output-fönstret:**
   - View → Output
   - Välj "Debug" i dropdown-listan

3. **Aktivera verbose output:**
   - Tools → Options → Debugging → Output Window
   - Sätt "Module Load Messages" till "On"

## Viktiga filer

- **Debug DLL:** `revit-ifc\Source\Revit.IFC.Export\bin\x64\Debug\Revit.IFC.Export.dll`
- **Add-in DLL:** `C:\ProgramData\Autodesk\ApplicationPlugins\IFC 2026.bundle\Contents\2026\Revit.IFC.Export.dll`
- **Installerad DLL:** `C:\Program Files\Autodesk\Revit 2026\Revit.IFC.Export.dll`

## Nästa steg för att lösa export-felet

Eftersom exporten fortfarande kraschar med "unrecoverable error", behöver vi:

1. **Debugga export-processen** för att se exakt var felet uppstår
2. **Kontrollera journal filen** efter varje export-försök
3. **Verifiera att din custom DLL faktiskt används** (kolla timestamp i Output-fönstret)

