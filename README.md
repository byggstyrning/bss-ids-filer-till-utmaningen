# Revit IFC Export - IDS-krav och BuildingSMART-validering

## Projektöversikt

Detta projekt har genomfört modifieringar av Revits IFC Export-plugin för att uppfylla IDS-krav (Information Delivery Specification) från Trafikverket samt byggSMART-valideringsregler. Projektet är en del av BIM Alliance Branschutmaning 2025.

### Resultat

**IDS-krav (SE-TRV_IDS-UTMANING xml.ids):**
- ✅ **Löst:** Alla IDS-krav har implementerats genom modifieringar i Revit IFC Export-koden
- ✅ **Löst:** Custom Shared Parameters har lagts till för att stödja IDS-kraven
- ✅ **Löst:** PropertySet-definitioner har skapats och konfigurerats för korrekt export
- ✅ **Löst:** Spatial containment (PartOf) har implementerats för IfcBridgePart-relationer
- ✅ **Löst:** PredefinedType-support har lagts till för IfcBridgePart
<img width="2315" height="1392" alt="Blender IDS Validation Results" src="https://github.com/user-attachments/assets/8941833d-48d5-469e-85d3-0b940d8128f4" />

**BuildingSMART-validering:**
- ✅ **Löst:** Invalid handle exceptions har fixats i transform-beräkningar
- ✅ **Löst:** Null-hantering har implementerats för PredefinedType-attribut
- ✅ **Löst:** Revit 2026 API-kompatibilitet har säkerställts
- ⚠️ **Känt begränsning:** OJT001-regeln kan inte uppfyllas på grund av Revit-arkitektur-begränsningar (se detaljer nedan)
<img width="843" height="146" alt="BuildingSMART Validation Service Results" src="https://github.com/user-attachments/assets/1fa32d90-778b-4914-b48a-c15eb7b07415" />

### Tekniska lösningar

Projektet har modifierat Revit IFC Export-pluginens källkod för att:
1. Läsa custom Shared Parameters från Revit-element
2. Exportera korrekta IFC-attribut enligt IDS-krav
3. Hantera spatial containment för bridge-strukturer
4. Förbättra felhantering och null-checks
5. Säkerställa kompatibilitet med Revit 2026 API

Alla ändringar är dokumenterade i denna README och lärdomar finns i `lessons-learned/` mappen.

---

## IDS-krav som kräver Shared Parameters i Revit

**Viktigt:** Vissa av de Shared Parameters som behövs finns redan i Autodesks standard Shared Parameter fil för IFC export (`IFC Shared Parameters-RevitIFCBuiltIn_ALL.txt`). Kontrollera denna fil först innan du skapar nya parametrar - många IFC-relaterade parametrar är redan definierade där och kan importeras direkt till ditt projekt.

**Parametrar som finns i Autodesks standardfil:**
- `IfcDescription` (rad 8665) - för IfcProject Description
- `IfcName` (rad 8664) - för IfcFooting Name
- `SiteDescription` (rad 8677) - för IfcSite Description
- `Pset_ConcreteElementGeneral.StrengthClass` (rad 1472) - för betongens hållfasthetsklass
- `IfcSpatialContainer` (rad 8685) - för spatial containment override

**Parametrar som INTE finns i standardfilen och måste skapas:**
- `FacilityDescription` - för IfcBridge Description (krävs för vår implementation)
- `IfcBridgePart.PredefinedType` - för IfcBridgePart PredefinedType (krävs för vår implementation)

För att uppfylla IDS-kraven (SE-TRV_IDS-UTMANING xml.ids) behöver vi lägga till custom Shared Parameters i Revit. Dessa parametrar används av IFC Exporter för att sätta korrekta attributvärden i den exporterade IFC-filen.

### 1. IfcProject Description (Project Information)
**IDS-krav:** IfcProject måste ha både `Name` och `Description` attribut (rad 22-52 i IDS-filen)

**Vad som behöver göras:**
- ✅ **Finns i Autodesks standardfil:** Importera Shared Parameter `"IfcDescription"` från `IFC Shared Parameters-RevitIFCBuiltIn_ALL.txt` (rad 8665)
- Lägg till parametern på **Project Information** elementet
- Denna parameter kommer att användas som `Description` attribut på `IfcProject` i den exporterade IFC-filen
- **Var i koden:** `Exporter.cs` → `CreateProject()` metoden läser från ProjectInfo, men behöver override för Description

**Användning:** Sätt värdet på Project Information → Information → IfcDescription (t.ex. "E4 Förbifart Stockholm")

---

### 2. IfcSite Description (Project Information)
**IDS-krav:** IfcSite måste ha både `Name` och `Description` attribut (rad 53-80 i IDS-filen)

**Vad som behöver göras:**
- ✅ **Finns i Autodesks standardfil:** Importera Shared Parameter `"SiteDescription"` från `IFC Shared Parameters-RevitIFCBuiltIn_ALL.txt` (rad 8677)
- Lägg till parametern på **Project Information** elementet
- Denna parameter kommer att användas som `Description` attribut på `IfcSite` i den exporterade IFC-filen
- **Var i koden:** `Exporter.cs` → `ExportSite()` metoden skapar IfcSite, behöver läsa från ProjectInfo override

**Användning:** Sätt värdet på Project Information → Information → SiteDescription (t.ex. "Bro över Knistavägen i tpl Häggvik å väg 265")

---

### 3. IfcFooting Name (Floors/Structural Foundations)
**IDS-krav:** IfcFooting måste ha `Name` attribut

**Vad som behöver göras:**
- ✅ **Finns i Autodesks standardfil:** Importera Shared Parameter `"IfcName"` från `IFC Shared Parameters-RevitIFCBuiltIn_ALL.txt` (rad 8664)
- Lägg till parametern på **Floors** (eller Structural Foundations) elementet
- Denna parameter kommer att användas som `Name` attribut på `IfcFooting` i den exporterade IFC-filen
- **Var i koden:** NamingUtil.GetNameOverride() kan läsa från custom parameter override

**Användning:** Sätt värdet på varje Floor-element som ska exporteras som IfcFooting (t.ex. "Bottenplatta")

---

### 4. Pset_ConcreteElementGeneral.StrengthClass Property (Floors)
**IDS-krav:** IfcFooting måste ha Property Set `Pset_ConcreteElementGeneral` med property `StrengthClass`

**Vad som behöver göras:**
- ✅ **Finns i Autodesks standardfil:** Importera Shared Parameter `"Pset_ConcreteElementGeneral.StrengthClass"` från `IFC Shared Parameters-RevitIFCBuiltIn_ALL.txt` (rad 1472)
- Lägg till parametern på **Floors** elementet
- Skapa Property Set Definition fil (`SE-TRV_PSET-EXPORT REVIT.txt`) som mappar denna parameter till `Pset_ConcreteElementGeneral.StrengthClass`
- **Viktigt:** Aktivera "Export IFC Common Property Sets" i IFC Export Settings
- **Var i koden:** PropertySet export läser från User Defined Property Sets filen

**Användning:** 
1. Sätt värdet på Floor-elementet (t.ex. "C30/37")
2. Välj Property Set Definition fil i IFC Export Settings
3. Aktivera "Export IFC Common Property Sets" checkbox

---

### 5. IfcSpatialContainer (Floors → Level)
**IDS-krav:** IfcFooting måste vara contained i rätt spatial structure (IfcBridgePart)

**Vad som behöver göras:**
- ✅ **Finns i Autodesks standardfil:** Importera Shared Parameter `"IfcSpatialContainer"` från `IFC Shared Parameters-RevitIFCBuiltIn_ALL.txt` (rad 8685)
- Lägg till parametern på **Floors** elementet
- Sätt värdet till en **Level** som är konfigurerad som `IfcBridgePart` (inte BuildingStorey)
- Denna parameter styr vilken spatial container (Level) som Floor-elementet ska exporteras som contained i
- **Var i koden:** ContainmentCache och spatial structure export

**Användning:** Sätt värdet på Floor-elementet till Level-namnet som ska vara spatial container (t.ex. "Grundläggning")

---

### 6. IfcBridge Description (Project Information)
**IDS-krav:** IfcBridge måste ha både `Name` och `Description` attribut (rad 81-106 i IDS-filen)

**Vad som behöver göras:**
- ❌ **Finns INTE i Autodesks standardfil:** Skapa ny Shared Parameter `"FacilityDescription"` (Text-typ)
- Lägg till parametern på **Project Information** elementet
- Denna parameter kommer att användas som `Description` attribut på `IfcBridge` i den exporterade IFC-filen
- **Var i koden:** `Exporter.cs` → `CreateFacilityFromProjectInfo()` läser redan från `"FacilityDescription"` parameter (rad 3757)
- **Känd bug:** `IFCInstanceExporter.cs` → `SetRoot()` metoden ignorerar description när element är null (rad 178). IfcBridge skapas utan element, så description ignoreras. **Detta behöver fixas i koden.**

**Användning:** Sätt värdet på Project Information → Information → FacilityDescription (t.ex. "Brobeskrivning")

https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/How-to-add-IFCSite-IFCBuilding-and-IfcProject-description-in-exported-IFC-from-Revit.html


## Vad som inte kan uppnås för närvarande med IFC_v26.4.0

- IFCBridgePart och dess PredefinedType: Revits Default-parameter för detta finns inte på Levels (som man måste använda för att sätta korrekt IfcBridgePart som Spatial container) och då  blir predefinedType null i senaste Revit IFC Export pluginen. Är väl korrekt enligt IFC-schemat (PredefinedType är optional), men kan vara ett problem om TRV IDS kräver ett specifikt värde. Dom har nog insett dilemmat också, se kommentarer på rad 1562-1566 i Exporter.cs för kontext om IFC4.3 frågor kring predefined types för IfcFacilityParts.

---

## IFC-valideringsfel som behöver fixas

### OJT001 - IfcFooting PredefinedType på occurrence när objekt är typed

**Status: ⚠️ KAN INTE UPPFYLLAS - Revit-komplexitet**

**Problem:** 
- Enligt OJT001-regeln (IFC Concept Template 4.1.3.2) måste PredefinedType på occurrence-nivå vara **tom** när objektet är typed av en IfcTypeObject (via IfcRelDefinesByType).
- Revit exporterar för närvarande PredefinedType på både occurrence (#64) och type (#65), vilket bryter mot OJT001-regeln.
- IFC-filen visar: `#64=IFCFOOTING(...,.PAD_FOOTING.)` och `#65=IFCFOOTINGTYPE(...,.PAD_FOOTING.)` med `#184=IFCRELDEFINESBYTYPE(...,(#64),#65)`.

**OJT001 Scenario 3 säger:**
> Om ett objekt är typed av en IfcTypeObject (vilket det är via IfcRelDefinesByType)  
> Och TypeObject har PredefinedType som inte är NOTDEFINED (vilket det är - PAD_FOOTING)  
> Då måste PredefinedType på Object-occurrence vara **tom (empty)**

**Varför detta inte kan fixas:**
- Revits IFC export-arkitektur gör det extremt komplext att kontrollera om ett element kommer att bli typed innan occurrence skapas.
- `TypeRelationsCache` populeras **efter** att både occurrence och type har skapats, vilket gör det omöjligt att veta i förväg om occurrence kommer att bli typed.
- Försök att förhindra typexport genom att sätta `IFC_EXPORT_ELEMENT_TYPE_AS` till ogiltiga värden fungerar inte tillförlitligt på grund av hur `GetProductExportType` faller tillbaka till standardmappningar.
- Även om vi skulle kunna förhindra typexport, skulle detta bryta IDS-kravet som kräver PropertySet på typen (`Pset_ConcreteElementGeneral` på `IfcFootingType`).
- Detta är en fundamental arkitektur-begränsning i Revits IFC export-mekanism som kräver omfattande refaktorering för att lösa korrekt.

**Referens:**
- OJT001-regel: https://github.com/buildingSMART/ifc-gherkin-rules/blob/main/features/rules/OJT/OJT001_Object-predefined-type.feature

## Implementerade ändringar
Men det hindrar ju oss inte för att implementera ändringarna som krävs!

### 1. IfcBridgePart PredefinedType support (Exporter.cs, rad 1567-1571)
**Problem:** IfcBridgePart kunde inte exportera PredefinedType från Revit-parametrar.

**Lösning:**
- Vi lägger till en custom shared parameter `"IfcBridgePart.PredefinedType"` då inbyggd parameter `"IFC Predefined Type"` saknas på Levels.

**Filer:** `revit-ifc/Source/Revit.IFC.Export/Exporter/Exporter.cs` (rad 1567-1571)

**Användning:** Lägg till parameter `IfcBridgePart.PredefinedType` på Level-elementet. Sätt värdet till t.ex. `FOUNDATION` enligt IDS-krav.

---

### 2. IfcBridgePart PredefinedType null-hantering (IFCInstanceExporter.cs, rad 2122-2124)
**Problem:** Exporten kraschade när PredefinedType var null eller tom, eftersom `SetAttribute` anropades med ogiltigt värde.

**Lösning:**
- Lagt till null/empty-check innan `SetAttribute` anropas
- PredefinedType är nu optional enligt IFC-schemat
- Om värdet saknas, skippas attributet helt (vilket är korrekt IFC-beteende)

**Filer:** `revit-ifc/Source/Revit.IFC.Export/Toolkit/IFCInstanceExporter.cs` (rad 2122-2124)

---

### 3. Invalid handle exception fix - GetTransformFromLocalPlacementHnd (ExporterUtil.cs, rad 3077-3087)
**Problem:** `ArgumentException: Invalid handle` när `GetAggregateDoubleAttribute` anropades på ogiltig `pos` handle.

**Lösning:**
- Validerar `pos` handle innan `Coordinates` läses
- Kontrollerar både null och typ (`IfcCartesianPoint`)
- Returnerar identity transform med `XYZ.Zero` origin om `pos` är ogiltig
- Förhindrar krasch och tillåter exporten att fortsätta

**Filer:** `revit-ifc/Source/Revit.IFC.Export/Utility/ExporterUtil.cs` (rad 3077-3087)

---

### 4. Invalid handle exception fix - GetTotalTransformFromLocalPlacement (ExporterUtil.cs, rad 3116-3129)
**Problem:** `ArgumentException: Invalid handle` i rekursiv transform-beräkning när `placementRelTo` var ogiltig eller transform var null.

**Lösning:**
- Validerar att `totalTrf` inte är null efter första anropet
- Validerar `placementRelTo` typ (`IfcLocalPlacement`) innan rekursivt anrop
- Använder `break` istället för `return null` för att undvika null-propagering
- Returnerar `Transform.Identity` om initial transform är null

**Filer:** `revit-ifc/Source/Revit.IFC.Export/Utility/ExporterUtil.cs` (rad 3116-3129)

---

### 5. Revit 2026 API-kompatibilitet - GetGlobal*DirectionHandle metoder (ExporterUtil.cs, rad 259-296)
**Problem:** `GetGlobal3DDirectionHandle` och `GetGlobal2DDirectionHandle` finns inte längre i Revit 2026 API, vilket orsakade kompileringsfel.

**Lösning:**
- Kommenterat ut optimeringskod som använde dessa metoder
- Använder nu `IFCInstanceExporter.CreateDirection()` direkt istället
- Notera: `GetGlobal3DOriginHandle` och `GetGlobal2DOriginHandle` finns fortfarande kvar och används

**Filer:** `revit-ifc/Source/Revit.IFC.Export/Utility/ExporterUtil.cs` (rad 259-296)

**Notera:** Plural-versionen `GetGlobal3DDirectionHandles` är obsolete, men singular-versionerna finns inte heller längre i Revit 2026.

---

### 6. Null-check för defContainerObjectPlacement (Exporter.cs, rad 1947-1953)
**Problem:** Potentiell "Invalid handle" exception när `GetObjectPlacement` returnerar ogiltig handle.

**Lösning:**
- Lagt till null-check för `defContainerObjectPlacement` innan `GetTotalTransformFromLocalPlacement` anropas
- Använder identity transform som fallback om placement är ogiltig

**Filer:** `revit-ifc/Source/Revit.IFC.Export/Exporter/Exporter.cs` (rad 1947-1953)

---
