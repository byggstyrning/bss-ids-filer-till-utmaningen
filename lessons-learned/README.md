# Lessons Learned - Revit IFC Export Modifications

Denna mapp innehåller dokumentation och lärdomar från arbetet med att modifiera Revit IFC Export för att uppfylla IDS-krav.

## Struktur

### 📁 bugfixes/
Dokumentation om specifika bugfixar som implementerats:
- `BUGFIX_PREDEFINEDTYPE_NULL.md` - Fix för null-hantering av PredefinedType
- `GETTOTALTRANSFORM_FIX.md` - Fix för transform-beräkningar
- `INVALID_HANDLE_FIX.md` - Fix för invalid handle exceptions

### 📁 analysis/
Analysdokument om IFC-strukturer och implementationer:
- `IFCBridge_Name_Description_Analysis.md` - Analys av IfcBridge Name/Description
- `IFCBridgePart_PredefinedType_Analysis.md` - Analys av IfcBridgePart PredefinedType
- `IFCBridgePart_PredefinedType_Implementation_Guide.md` - Implementation guide för PredefinedType
- `PartOf_Relationships_Analysis.md` - Analys av PartOf-relationer
- `PredefinedType_Best_Practices_Analysis.md` - Best practices för PredefinedType
- `things we need to update.md` - Valideringsfel och lösningar

### 📁 build-and-installation/
Dokumentation om build-process och installation:
- `BUILD_GUIDE.md` - Guide för att bygga projektet
- `BUILD_PLAN.md` - Plan för build-processen
- `BUILD_STATUS.md` - Status för build-processen
- `BUILD_SUCCESS_SUMMARY.md` - Sammanfattning av lyckad build
- `INSTALLATION_COMPLETE.md` - Installation completion notes
- `DLL_LOCATION.md` - Information om DLL-platser

### 📁 debugging/
Guider för debugging och troubleshooting:
- `DEBUGGING_GUIDE.md` - Allmän debugging guide
- `EXCEPTION_DEBUGGING_GUIDE.md` - Guide för exception debugging

## Viktiga lärdomar

### Revit IFC Export Arkitektur
- `TypeRelationsCache` populeras **efter** att både occurrence och type har skapats
- Det är omöjligt att veta i förväg om ett element kommer att bli typed
- `GetProductExportType()` faller tillbaka till standardmappningar även när parametrar är ogiltiga

### OJT001-validering
- OJT001-regeln kan inte uppfyllas för typed objects i Revit-exporten på grund av arkitektur-begränsningar
- Detta är ett känt begränsning som kräver ändringar på Revit-sidan

### PropertySet Export
- PropertySets kan exporteras till både instanser (`I`) och typer (`T`)
- IDS-krav kan kräva PropertySets på instanser även när typer exporteras
- PropertySet Definition-filer (`*.txt`) styr var PropertySets exporteras

## Se även

- `../TODO.md` - Aktuella uppgifter och implementerade ändringar
- `../SE-TRV_IDS-UTMANING xml.ids` - IDS-krav som ska uppfyllas
- `../SE-TRV_PSET-EXPORT REVIT.txt` - PropertySet Definition fil

