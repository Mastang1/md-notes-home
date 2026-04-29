[!/* *************************************************************************
* File: gen_version.m
* Description: Extract version info from XDM CommonPublishedInformation
*************************************************************************
*/!]

[!/* ====================================================================
     方式一：提取为全局变量，方便在任意文件的任意位置灵活组合 
   ==================================================================== */!]
[!VAR "Ipcf_VendorId"       = "num:i(CommonPublishedInformation/VendorId)"!]
[!VAR "Ipcf_ModuleId"       = "num:i(CommonPublishedInformation/ModuleId)"!]
[!VAR "Ipcf_ArReleaseMajor" = "num:i(CommonPublishedInformation/ArReleaseMajorVersion)"!]
[!VAR "Ipcf_ArReleaseMinor" = "num:i(CommonPublishedInformation/ArReleaseMinorVersion)"!]
[!VAR "Ipcf_ArReleaseRev"   = "num:i(CommonPublishedInformation/ArReleaseRevisionVersion)"!]
[!VAR "Ipcf_SwMajor"        = "num:i(CommonPublishedInformation/SwMajorVersion)"!]
[!VAR "Ipcf_SwMinor"        = "num:i(CommonPublishedInformation/SwMinorVersion)"!]
[!VAR "Ipcf_SwPatch"        = "num:i(CommonPublishedInformation/SwPatchVersion)"!]


[!/* ====================================================================
     方式二：封装为代码块宏，方便在头文件中一键生成整段 #define
   ==================================================================== */!]
[!MACRO "Ipcf_GenerateVersionMacros"!]
/* AUTOSAR Specification Version Information */
#define IPCF_IP_CFG_DEFINES_VENDOR_ID                    [!"$Ipcf_VendorId"!]
#define IPCF_IP_CFG_DEFINES_MODULE_ID                    [!"$Ipcf_ModuleId"!]
#define IPCF_IP_CFG_DEFINES_AR_RELEASE_MAJOR_VERSION     [!"$Ipcf_ArReleaseMajor"!]
#define IPCF_IP_CFG_DEFINES_AR_RELEASE_MINOR_VERSION     [!"$Ipcf_ArReleaseMinor"!]
#define IPCF_IP_CFG_DEFINES_AR_RELEASE_REVISION_VERSION  [!"$Ipcf_ArReleaseRev"!]

/* Software Version Information */
#define IPCF_IP_CFG_DEFINES_SW_MAJOR_VERSION             [!"$Ipcf_SwMajor"!]
#define IPCF_IP_CFG_DEFINES_SW_MINOR_VERSION             [!"$Ipcf_SwMinor"!]
#define IPCF_IP_CFG_DEFINES_SW_PATCH_VERSION             [!"$Ipcf_SwPatch"!]
[!ENDMACRO!]