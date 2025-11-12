# IFCBridgePart PredefinedType Implementation Guide

## Code Change Completed ✅

The code has been modified in:
- **File**: `revit-ifc/Source/Revit.IFC.Export/Exporter/Exporter.cs`
- **Line**: 1568
- **Change**: Now reads predefinedType from Level parameter instead of hardcoded `null`

```csharp
// BEFORE:
string predefinedType = null;

// AFTER:
string predefinedType = NamingUtil.GetOverrideStringValue(level, "IfcBridgePart.PredefinedType", null);
```

## Next Steps

### 1. Rebuild the Solution

You need to rebuild the Revit IFC exporter solution:
1. Open `revit-ifc/Revit.IFC.sln` in Visual Studio
2. Select the correct Revit version configuration (e.g., Release_25.x.x for Revit 2025)
3. Build the solution: `Build > Build Solution`
4. Copy the resulting DLLs to the add-in folder (typically: `C:\ProgramData\Autodesk\ApplicationPlugins\IFC 2025.bundle\Contents\2025`)

### 2. Add Shared Parameter to Levels

**Important**: The parameter name and value must match exactly:

1. **Create/Edit Shared Parameter File**:
   - Open Revit
   - Manage → Shared Parameters
   - Create or edit your shared parameter file

2. **Add Parameter**:
   - **Parameter Name**: `IfcBridgePart.PredefinedType` (exact, case-sensitive)
   - **Parameter Type**: Text
   - **Group**: IFC Data (or your preferred group)
   - **Instance** (not Type)

3. **Add Parameter to Level Category**:
   - Manage → Project Parameters
   - Add → Select "Shared parameter"
   - Choose `IfcBridgePart.PredefinedType`
   - Check "Levels" category
   - OK

4. **Set Parameter Value**:
   - Open a Level in your project
   - Find the parameter `IfcBridgePart.PredefinedType`
   - Set value to: `FOUNDATION` (exact, all caps, no quotes)

### 3. Valid PredefinedType Values

The value must match exactly (case-sensitive) one of these enum values:

- `FOUNDATION` ← **This is what you need**
- `ABUTMENT`
- `DECK`
- `DECK_SEGMENT`
- `PIER`
- `PIER_SEGMENT`
- `PYLON`
- `SUBSTRUCTURE`
- `SUPERSTRUCTURE`
- `SURFACESTRUCTURE`
- `USERDEFINED`
- `NOTDEFINED`

### 4. Test the Export

1. Set the parameter value on your Level(s)
2. Export IFC:
   - File → Export → IFC
   - Select IFC4x3
   - Set Facility Type to "Bridge"
   - Export
3. Check the exported IFC file:
   - Open the `.ifc` file in a text editor
   - Find your `IFCBRIDGEPART` entity
   - Verify the PredefinedType attribute shows `.FOUNDATION.` (with dots)

## Troubleshooting

### Parameter Not Found
- **Check**: Parameter name is exactly `IfcBridgePart.PredefinedType` (case-sensitive)
- **Check**: Parameter is added to "Levels" category
- **Check**: Parameter is Instance (not Type)

### Value Not Exporting
- **Check**: Value is exactly `FOUNDATION` (all caps, no spaces)
- **Check**: Code has been rebuilt and DLLs copied to add-in folder
- **Check**: You're exporting as IFC4x3 (not older versions)

### Still Shows `$` (null) in IFC
- **Check**: Parameter value is set on the Level element
- **Check**: Parameter name matches exactly: `IfcBridgePart.PredefinedType`
- **Check**: Rebuild and restart Revit after code changes

## Example IFC Output

**Before** (with null):
```
#36=IFCBRIDGEPART(...,.ELEMENT.,$,$);
```

**After** (with FOUNDATION):
```
#36=IFCBRIDGEPART(...,.ELEMENT.,.FOUNDATION.,$);
```

## Related Files Modified

- `revit-ifc/Source/Revit.IFC.Export/Exporter/Exporter.cs` (line 1568)

## Notes

- The parameter name follows the pattern: `"IfcEntityName.AttributeName"` (same as `"IfcRampFlight.PredefinedType"`)
- The value must match the enum exactly (case-sensitive)
- If parameter is not found or empty, it will export as `null` (`$` in IFC)

