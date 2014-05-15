'''
Name:
Purpose:

Author:      Mike Jones

Created:     31Mar2014
Copyright:   (c) Mike Jones 2014
Licence:
    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    The GNU General Public License can be found at
    &lt;http://www.gnu.org/licenses&gt;.
'''
import arcpy
import os
from arcpy import env


class Toolbox(object):
    def __init__(self):
        """Define the toolbox (the name of the toolbox is the name of the
        .pyt file)."""
        self.label = "Load Leica Data onto SDE"
        self.alias = "LeicaSurveytoSDE"

        # List of tool classes associated with this toolbox
        self.tools = [SMONs]

class SMONs(object):
    def __init__(self):
        """Define the tool (tool name is the name of the class)."""
        self.label = "SMONs to Database"
        self.description = "Joins tables and places the points in GDB"
        self.canRunInBackground = True

    def getParameterInfo(self):

        # First parameter
        shpGEO = arcpy.Parameter(
            displayName="Input points with Geometry.  I.E. SMON(2)",
            name="InputGDB",
            datatype="DEFeatureClass",
            parameterType="Required",
            direction="Input")

        # Set the filter to accept only local (personal or file) geodatabases
        shpGEO.filter.list = ["Local Database"]

        # Second Parameter
        shpTable = arcpy.Parameter(
            displayName="Input points with Northing, Easting, Elevation, and Point_ID.  I.E. SMON",
            name="Suffix",
            datatype="DEFeatureClass",
            parameterType="Required",
            direction="Input")

        # Third Parameter
        action = arcpy.Parameter(
            displayName="Control Point or SMON?",
            name="action",
            datatype="String",
            parameterType="Required",
            direction="Input")

        # Set a value list of Control Point or Smon
        action.filter.type = "ValueList"
        action.filter.list = ["Control Point", "SMON"]

        params = [shpGEO, shpTable, action]
        return params

    def isLicensed(self):
        """Set whether tool is licensed to execute."""
        return True

    def updateParameters(self, parameters):
        """Modify the values and properties of parameters before internal
        validation is performed.  This method is called whenever a parameter
        has been changed."""
        return

    def updateMessages(self, parameters):
        """Modify the messages created by internal validation for each tool
        parameter.  This method is called after internal validation."""

        return

    def execute(self, parameters, messages):
        """The source code of the tool."""
        # Get inputs
        shpGEO = parameters[0].valueAsText
        shpTable = parameters[1].valueAsText
        action = parameters[2].valueAsText

        #Set Date
        timestr = time.strftime("%Y_%m_%d_%H%M")

        # Set workspace as defined by user
        arcpy.workspace = shpGEO
        arcpy.env.overwriteOutput = True
        # Rename Feature Classes

        if action == "Control Point":
            arcpy.env.overwriteOutput = True
            #Archive
            arcpy.CopyFeatures_management(shpGEO, r"Q:\GIS\PAM_STATE_ARCHIVE\Leica" + os.sep + "SMON" + time.strftime('_%y_%m_%d_%H%M'))
            arcpy.CopyFeatures_management(shpTable, r"Q:\GIS\PAM_STATE_ARCHIVE\Leica" + os.sep + "SMON2" + time.strftime('_%y_%m_%d_%H%M'))
            arcpy.AddMessage("Shapes Archived")

            #Copy data to working file
            arcpy.CopyFeatures_management(shpGEO, r"P:\Scripts\SystemData\Scratch\shpGEO.shp" + timestr)
            arcpy.CopyFeatures_management(shpTable,r"P:\Scripts\SystemData\Scratch\shpTable.shp" + timestr)
            arcpy.AddMessage("Working File Created")

            #Define working files
            wrkSHP = r"P:\Scripts\SystemData\Scratch\shpGEO.shp"
            wrkTable = r"P:\Scripts\SystemData\Scratch\shpTable.shp"
            wrkSHP_83 = r"P:\Scripts\SystemData\Scratch\shpGEO_83.shp"
            #Join Tables
            arcpy.JoinField_management(wrkSHP, "Point_ID", wrkTable, "Point_ID", "Easting_Lo;Northing_L;Ortho_Heig")
            arcpy.AddMessage("Tables Joined")

            #Add Fields
            arcpy.AddField_management(wrkSHP, 'shpname','text')
            arcpy.AddField_management(wrkSHP, 'GIS_ID_PAM','text')
            arcpy.AddField_management(wrkSHP, 'LATDMS','text')
            arcpy.AddField_management(wrkSHP, 'LONGDMS','text')
            arcpy.AddField_management(wrkSHP, 'DESC_','text')
            arcpy.AddField_management(wrkSHP, 'StatePlane','text')
            arcpy.AddField_management(wrkSHP, 'State','text')
            arcpy.AddField_management(wrkSHP, 'Unique_ID','text')
            arcpy.AddField_management(wrkSHP, 'DateImport','text')
            arcpy.AddMessage("Fields Added")

            #Calc Fields
            arcpy.CalculateField_management(wrkSHP, "LATDMS", "decdeg2dms ( !Northing_L! )", "PYTHON", "def decdeg2dms(dd):\\n    negative = dd < 0\\n    dd = abs(dd)\\n    minutes,seconds =  divmod(dd*3600,60)\\n    degrees,minutes =   divmod(minutes,60)\\n    if negative:\\n        if degrees > 0:\\n            degrees = -degrees\\n        elif minutes > 0:\\n            minutes = -minutes\\n        else:\\n            seconds = -seconds\\n            seconds = round(seconds, 5) #my attempt to define it\\n   \\n    dms = '{0:.0f} {1:.0f} {2:.5f}'.format(degrees, minutes, seconds)\\n    \\n    return (dms)")
            arcpy.CalculateField_management(wrkSHP, "LONGDMS", "decdeg2dms ( !Easting_Lo! )", "PYTHON", "def decdeg2dms(dd):\\n    negative = dd < 0\\n    dd = abs(dd)\\n    minutes,seconds =  divmod(dd*3600,60)\\n    degrees,minutes =   divmod(minutes,60)\\n    if negative:\\n        if degrees > 0:\\n            degrees = -degrees\\n        elif minutes > 0:\\n            minutes = -minutes\\n        else:\\n            seconds = -seconds\\n            seconds = round(seconds, 5) #my attempt to define it\\n   \\n    dms = '{0:.0f} {1:.0f} {2:.5f}'.format(degrees, minutes, seconds)\\n    \\n    return (dms)")
            urows = arcpy.UpdateCursor(wrkSHP)
            for urow in urows:
                urow.DateImport = str(timestr)
                urow.GIS_ID_PAM = "PAM - " + urow.Point_ID
                urow.shpname = "Leica"
                urow.DESC_ = urow.Point_ID + urow.DateEstabl + urow.Monument_T + urow.Monument_S + urow.Collection
                urow.StatePlane = "Colorado North"
                urow.State = "Colorado"
                urow.Unique_ID = str(urow.Point_ID) + "0501"
                urows.updateRow(urow)
            arcpy.AddMessage("Fields Calculated")

            # Define Projection
            arcpy.DefineProjection_management(wrkSHP, "GEOGCS['GCS_WGS_1984',DATUM['D_WGS_1984',SPHEROID['WGS_1984',6378137.0,298.257223563]],PRIMEM['Greenwich',0.0],UNIT['Degree',0.0174532925199433]],VERTCS['TBC_VCS',VDATUM['GEOID12A_(Conus)'],PARAMETER['Vertical_Shift',0.0],PARAMETER['Direction',1.0],UNIT['Foot_US',0.3048006096012192]]")
            arcpy.AddMessage("Projection Defined")
            # Project to NAD 83 from WGS84
            arcpy.Project_management(wrkSHP, wrkSHP_83, "GEOGCS['GCS_North_American_1983',DATUM['D_North_American_1983',SPHEROID['GRS_1980',6378137.0,298.257222101]],PRIMEM['Greenwich',0.0],UNIT['Degree',0.0174532925199433]]", "WGS_1984_(ITRF00)_To_NAD_1983", "GEOGCS['GCS_WGS_1984',DATUM['D_WGS_1984',SPHEROID['WGS_1984',6378137.0,298.257223563]],PRIMEM['Greenwich',0.0],UNIT['Degree',0.0174532925199433]],VERTCS['TBC_VCS',VDATUM['GEOID12A_(Conus)'],PARAMETER['Vertical_Shift',0.0],PARAMETER['Direction',1.0],UNIT['Foot_US',0.3048006096012192]]")
            arcpy.AddMessage("Projected to NAD83 from WGS84")

            # Append to SDE database
            sdePath = r"C:\Users\jjones\AppData\Roaming\ESRI\Desktop10.1\ArcCatalog\PFS_Survey.sde"
            sdeCP = r"Client_PFS_Survey.DBO.ControlPoints_NewSchema"
            cpOutput = sdePath + os.sep + sdeCP

            arcpy.Append_management(wrkSHP_83, cpOutput, "NO_TEST", "Monument_T \"Monument_T\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Monument_T,-1,-1;Monument_S \"Monument_S\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Monument_S,-1,-1;Collection \"Collection\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Collection,-1,-1;DateEstabl \"DateEstabl\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,DateEstabl,-1,-1;Name \"Name\" true true false 30 Text 0 0 ,First,#;Comments \"Comments\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Comments,-1,-1;PointID \"PointID\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Point_ID,-1,-1;Easting \"Easting\" true true false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Easting__1,-1,-1;Northing \"Northing\" true true false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Northing_1,-1,-1;Elevation \"Elevation\" true true false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Ortho_Heig,-1,-1;GlobalLati \"GlobalLati\" true true false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Northing_L,-1,-1;GlobalLong \"GlobalLong\" true true false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Easting_Lo,-1,-1;GlobalElli \"GlobalElli\" true true false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,WGS84_Elli,-1,-1;LocalLatit \"LocalLatit\" true true false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Northing_L,-1,-1;LocalLongi \"LocalLongi\" true true false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Easting_Lo,-1,-1;LocalEllip \"LocalEllip\" true true false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,WGS84_Elli,-1,-1;shpname \"shpname\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,shpname,-1,-1;GIS_ID_PAM \"GIS_ID_PAM\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,GIS_ID_PAM,-1,-1;LATDMS \"LATDMS\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,LATDMS,-1,-1;LONGDMS \"LONGDMS\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,LONGDMS,-1,-1;DESC_ \"DESC_\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,DESC_,-1,-1;StatePlane \"StatePlane\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,StatePlane,-1,-1;State \"State\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,State,-1,-1;Unique_ID \"Unique_ID\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Unique_ID,-1,-1;DateImport \"DateImport\" true true false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,DateImport,-1,-1", "")
            open(r"Q:\FIELD_DATA_TO_PAM\FROM_LGO\COLORADO\CP_IMPORT" + timestr + ".txt", "w")
            arcpy.AddMessage("Added to Database!")


        ######################################################## SMONS  #########################################################
        elif action == "SMON":
            arcpy.env.overwriteOutput = True
           #Archive
            arcpy.CopyFeatures_management(shpGEO, r"Q:\GIS\PAM_STATE_ARCHIVE\Leica" + os.sep + "SMON" + time.strftime('_%y_%m_%d_%H%M'))
            arcpy.CopyFeatures_management(shpTable, r"Q:\GIS\PAM_STATE_ARCHIVE\Leica" + os.sep + "SMON2" + time.strftime('_%y_%m_%d_%H%M'))
            arcpy.AddMessage("Shapes Archived")

            #Copy data to working file
            arcpy.CopyFeatures_management(shpGEO, r"P:\Scripts\SystemData\Scratch\shpGEO.shp" + timestr)
            arcpy.CopyFeatures_management(shpTable,r"P:\Scripts\SystemData\Scratch\shpTable.shp" + timestr)
            arcpy.AddMessage("Working File Created")

            #Define working files
            wrkSHP = r"P:\Scripts\SystemData\Scratch\shpGEO.shp"
            wrkTable = r"P:\Scripts\SystemData\Scratch\shpTable.shp"
            wrkSHP_83 = r"P:\Scripts\SystemData\Scratch\shpGEO_83.shp"
            #Join Tables
            arcpy.JoinField_management(wrkSHP, "Point_ID", wrkTable, "Point_ID", "Easting_Lo;Northing_L;Ortho_Heig")
            arcpy.AddMessage("Tables Joined")

            #Add Fields
            arcpy.AddField_management(wrkSHP, 'shpname','text')
            arcpy.AddField_management(wrkSHP, 'GIS_ID_PAM','text')
            arcpy.AddField_management(wrkSHP, 'LATDMS','text')
            arcpy.AddField_management(wrkSHP, 'LONGDMS','text')
            arcpy.AddField_management(wrkSHP, 'DESC_','text')
            arcpy.AddField_management(wrkSHP, 'StatePlane','text')
            arcpy.AddField_management(wrkSHP, 'State','text')
            arcpy.AddField_management(wrkSHP, 'Unique_ID','text')
            arcpy.AddField_management(wrkSHP, 'DateImport','text')
            arcpy.AddMessage("Fields Added")

            #Calc Fields
            arcpy.CalculateField_management(wrkSHP, "LATDMS", "decdeg2dms ( !Northing_L! )", "PYTHON", "def decdeg2dms(dd):\\n    negative = dd < 0\\n    dd = abs(dd)\\n    minutes,seconds =  divmod(dd*3600,60)\\n    degrees,minutes =   divmod(minutes,60)\\n    if negative:\\n        if degrees > 0:\\n            degrees = -degrees\\n        elif minutes > 0:\\n            minutes = -minutes\\n        else:\\n            seconds = -seconds\\n            seconds = round(seconds, 5) #my attempt to define it\\n   \\n    dms = '{0:.0f} {1:.0f} {2:.5f}'.format(degrees, minutes, seconds)\\n    \\n    return (dms)")
            arcpy.CalculateField_management(wrkSHP, "LONGDMS", "decdeg2dms ( !Easting_Lo! )", "PYTHON", "def decdeg2dms(dd):\\n    negative = dd < 0\\n    dd = abs(dd)\\n    minutes,seconds =  divmod(dd*3600,60)\\n    degrees,minutes =   divmod(minutes,60)\\n    if negative:\\n        if degrees > 0:\\n            degrees = -degrees\\n        elif minutes > 0:\\n            minutes = -minutes\\n        else:\\n            seconds = -seconds\\n            seconds = round(seconds, 5) #my attempt to define it\\n   \\n    dms = '{0:.0f} {1:.0f} {2:.5f}'.format(degrees, minutes, seconds)\\n    \\n    return (dms)")
            urows = arcpy.UpdateCursor(wrkSHP)
            for urow in urows:
                urow.DateImport = str(timestr)
                urow.GIS_ID_PAM = "PAM - " + urow.Point_ID
                urow.shpname = "Leica"
                urow.DESC_ = urow.Point_ID + urow.Monument_S + urow.Monument_T + urow.TownshipRa + urow.Date_Shot + urow.Date_Shot
                urow.StatePlane = "Colorado North"
                urow.State = "Colorado"
                urow.Unique_ID = str(urow.Point_ID) + "0501"
                urows.updateRow(urow)
            arcpy.AddMessage("Fields Calculated")

            # Define Projection
            arcpy.DefineProjection_management(wrkSHP, "GEOGCS['GCS_WGS_1984',DATUM['D_WGS_1984',SPHEROID['WGS_1984',6378137.0,298.257223563]],PRIMEM['Greenwich',0.0],UNIT['Degree',0.0174532925199433]],VERTCS['TBC_VCS',VDATUM['GEOID12A_(Conus)'],PARAMETER['Vertical_Shift',0.0],PARAMETER['Direction',1.0],UNIT['Foot_US',0.3048006096012192]]")
            arcpy.AddMessage("Projection Defined")
            # Project to NAD 83 from WGS84
            arcpy.Project_management(wrkSHP, wrkSHP_83, "GEOGCS['GCS_North_American_1983',DATUM['D_North_American_1983',SPHEROID['GRS_1980',6378137.0,298.257222101]],PRIMEM['Greenwich',0.0],UNIT['Degree',0.0174532925199433]]", "WGS_1984_(ITRF00)_To_NAD_1983", "GEOGCS['GCS_WGS_1984',DATUM['D_WGS_1984',SPHEROID['WGS_1984',6378137.0,298.257223563]],PRIMEM['Greenwich',0.0],UNIT['Degree',0.0174532925199433]],VERTCS['TBC_VCS',VDATUM['GEOID12A_(Conus)'],PARAMETER['Vertical_Shift',0.0],PARAMETER['Direction',1.0],UNIT['Foot_US',0.3048006096012192]]")
            arcpy.AddMessage("Projected to NAD83 from WGS84")


            # Append to a SDE database
            sdePath = r"C:\Users\jjones\AppData\Roaming\ESRI\Desktop10.1\ArcCatalog\PFS_Survey.sde"
            sdeSMON = r"Client_PFS_Survey.DBO.SMONs_NewSchema"
            smonOutput = sdePath + os.sep + sdeSMON

            arcpy.Append_management(wrkSHP_83, smonOutput, "NO_TEST", "Type \"Type\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Monument_T,-1,-1;Mounment_S \"Mounment_S\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Monument_S,-1,-1;TownshipRa \"TownshipRa\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,TownshipRa,-1,-1;Index_ \"Index_\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Index_,-1,-1;Date_Shot \"Date_Shot\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Date_Shot,-1,-1;Date_PLS_ \"Date_PLS_\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Date_PLS_,-1,-1;Comments \"Comments\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Comments,-1,-1;PointID \"PointID\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Point_ID,-1,-1;Easting \"Easting\" true false false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Easting__1,-1,-1;Northing \"Northing\" true false false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Northing_1,-1,-1;Elevation \"Elevation\" true false false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Ortho_Heig,-1,-1;GlobalLati \"GlobalLati\" true false false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Northing_L,-1,-1;GlobalLong \"GlobalLong\" true false false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Easting_Lo,-1,-1;GlobalElli \"GlobalElli\" true false false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,WGS84_Elli,-1,-1;LocalLatit \"LocalLatit\" true false false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Northing_L,-1,-1;LocalLongi \"LocalLongi\" true false false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Easting_Lo,-1,-1;LocalEllip \"LocalEllip\" true false false 8 Double 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,WGS84_Elli,-1,-1;shpname \"shpname\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,shpname,-1,-1;GIS_ID_PAM \"GIS_ID_PAM\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,GIS_ID_PAM,-1,-1;LATDMS \"LATDMS\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,LATDMS,-1,-1;LONGDMS \"LONGDMS\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,LONGDMS,-1,-1;DESC_ \"DESC_\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,DESC_,-1,-1;StatePlane \"StatePlane\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,StatePlane,-1,-1;State \"State\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,State,-1,-1;Unique_ID \"Unique_ID\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,Unique_ID,-1,-1;DateImport \"DateImport\" true false false 254 Text 0 0 ,First,#,P:\\Scripts\\SystemData\\Scratch\\shpGEO_83.shp,DateImport,-1,-1", "")
            open(r"Q:\FIELD_DATA_TO_PAM\FROM_LGO\COLORADO\SMON_IMPORT" + timestr + ".txt", "w")
            arcpy.AddMessage("Added to Database!")

        return
