---
title: '【メモ】 Windows11とTiny11のサービス差分'
date: 2024-04-07T01:09:00+09:00
draft: false
categories: ["Windows"]
description:  
image: "img/スクリーンショット-2024-04-07-1.10.32.png"
# original: https://mikuta0407.net/wordpress/index.php/2024/04/07/win11_tiny11_services_diff/
---

個人用メモに近いですが、Windows11とNTDEV氏作成のTiny11のサービス差分を確認してみます。

## なんのために?

- RAM8GBのマシンにWindows11を入れようとした
- でも最近(といってももう10年くらい)のWindowsはリソース多めに食う
- 軽いTiny11を入れたらどうなるんだろう(VirtualBoxで検討)
- なんか500MB~600MBくらい差があるぞ
- VBox拡張だけ入れた状態で、RAM使用量が2.3GBと1.7GB
- サービスに差があるのかな?

## 確認に使用したもの

[Windowsの謎 ～サービス一覧を出すコマンド～ #Windows - Qiita](https://qiita.com/Mr-K/items/0ab787135eb4ec3b3a1c) を参考に表示カラムをちょい弄って実行

```powershell
$triggers = Get-ChildItem "HKLM:\SYSTEM\CurrentControlSet\Services" |
    Where-Object { $_.GetSubkeyNames().Contains("TriggerInfo") } |
    ForEach-Object { $_.Name.Split("\")[-1] }

$startMode = @{ Manual = "手動"; Disabled = "無効"; Auto = "自動"; Unknown = "不明" }
$startOption = @{ 01 = " (トリガー開始)"; 10 = " (遅延開始)"; 11 = " (遅延開始、トリガー開始)" }

$serviceData = Get-CimInstance -ClassName Win32_Service | Select-Object @(
    @{ n = "サービス名";          e = { $_.Name } }
    @{ n = "スタートアップの種類"; e = { $startMode[$_.StartMode] + $startOption[10 * ($_.StartMode -eq "Auto" -and $_.DelayedAutoStart) + $triggers.Contains($_.Name)] } }
    @{ n = "状態";                e = { if($_.State -eq "Running") { "実行" } else { "停止" } } }
)

$serviceData
```

## 一覧

<details>
<summary>純正</summary>

```
サービス名                               スタートアップの種類          状態
----------                               --------------------          ----
AJRouter                                 手動 (トリガー開始)           停止
ALG                                      手動                          停止
AppIDSvc                                 手動 (トリガー開始)           実行
Appinfo                                  手動 (トリガー開始)           実行
AppMgmt                                  手動                          停止
AppReadiness                             手動                          実行
AppVClient                               無効                          停止
AppXSvc                                  手動 (トリガー開始)           実行
AssignedAccessManagerSvc                 手動 (トリガー開始)           停止
AudioEndpointBuilder                     自動                          実行
Audiosrv                                 自動                          実行
autotimesvc                              手動 (トリガー開始)           停止
AxInstSV                                 手動                          停止
BDESVC                                   手動 (トリガー開始)           停止
BFE                                      自動                          実行
BITS                                     手動                          停止
BrokerInfrastructure                     自動                          実行
BTAGService                              手動 (トリガー開始)           停止
BthAvctpSvc                              手動 (トリガー開始)           停止
bthserv                                  手動 (トリガー開始)           停止
camsvc                                   手動 (トリガー開始)           実行
CDPSvc                                   自動 (遅延開始、トリガー開始) 実行
CertPropSvc                              手動 (トリガー開始)           停止
ClipSVC                                  手動 (トリガー開始)           実行
cloudidsvc                               手動                          停止
COMSysApp                                手動                          停止
CoreMessagingRegistrar                   自動                          実行
CryptSvc                                 自動 (トリガー開始)           実行
CscService                               手動 (トリガー開始)           停止
DcomLaunch                               自動                          実行
dcsvc                                    手動 (トリガー開始)           停止
defragsvc                                手動                          停止
DeviceAssociationService                 手動 (トリガー開始)           停止
DeviceInstall                            手動 (トリガー開始)           停止
DevQueryBroker                           手動 (トリガー開始)           停止
Dhcp                                     自動                          実行
diagnosticshub.standardcollector.service 手動                          停止
diagsvc                                  手動 (トリガー開始)           停止
DiagTrack                                自動                          実行
DialogBlockingService                    無効                          停止
DispBrokerDesktopSvc                     自動                          実行
DisplayEnhancementService                手動 (トリガー開始)           実行
DmEnrollmentSvc                          手動                          停止
dmwappushservice                         手動 (トリガー開始)           停止
Dnscache                                 自動 (トリガー開始)           実行
DoSvc                                    自動 (遅延開始、トリガー開始) 実行
dot3svc                                  手動                          停止
DPS                                      自動                          実行
DsmSvc                                   手動 (トリガー開始)           停止
DsSvc                                    手動 (トリガー開始)           停止
DusmSvc                                  自動                          実行
EapHost                                  手動                          停止
edgeupdate                               自動 (遅延開始、トリガー開始) 停止
edgeupdatem                              手動 (トリガー開始)           停止
EFS                                      手動 (トリガー開始)           停止
embeddedmode                             手動 (トリガー開始)           停止
EntAppSvc                                手動                          停止
EventLog                                 自動                          実行
EventSystem                              自動                          実行
fdPHost                                  手動                          停止
FDResPub                                 手動 (トリガー開始)           停止
fhsvc                                    手動 (トリガー開始)           停止
FontCache                                自動                          実行
FrameServer                              手動 (トリガー開始)           停止
FrameServerMonitor                       手動 (トリガー開始)           停止
GameInputSvc                             手動 (トリガー開始)           停止
gpsvc                                    自動 (トリガー開始)           実行
GraphicsPerfSvc                          手動 (トリガー開始)           停止
hidserv                                  手動 (トリガー開始)           停止
HvHost                                   手動 (トリガー開始)           停止
icssvc                                   手動 (トリガー開始)           停止
IKEEXT                                   手動 (トリガー開始)           停止
InstallService                           手動                          実行
InventorySvc                             手動                          停止
iphlpsvc                                 自動                          実行
IpxlatCfgSvc                             手動 (トリガー開始)           停止
KeyIso                                   手動 (トリガー開始)           実行
KtmRm                                    手動 (トリガー開始)           停止
LanmanServer                             自動 (トリガー開始)           実行
LanmanWorkstation                        自動                          実行
lfsvc                                    手動 (トリガー開始)           実行
LicenseManager                           手動 (トリガー開始)           実行
lltdsvc                                  手動                          停止
lmhosts                                  手動 (トリガー開始)           実行
LSM                                      不明                          実行
LxpSvc                                   手動                          停止
MapsBroker                               自動 (遅延開始)               停止
McpManagementService                     手動                          停止
MicrosoftEdgeElevationService            手動                          停止
MixedRealityOpenXRSvc                    手動                          停止
mpssvc                                   自動                          実行
MSDTC                                    手動                          停止
MSiSCSI                                  手動                          停止
msiserver                                手動                          停止
MsKeyboardFilter                         無効                          停止
NaturalAuthentication                    手動 (トリガー開始)           停止
NcaSvc                                   手動 (トリガー開始)           停止
NcbService                               手動 (トリガー開始)           実行
NcdAutoSetup                             手動 (トリガー開始)           停止
Netlogon                                 手動                          停止
Netman                                   手動                          停止
netprofm                                 手動                          実行
NetSetupSvc                              不明 (トリガー開始)           実行
NetTcpPortSharing                        無効                          停止
NgcCtnrSvc                               手動 (トリガー開始)           停止
NgcSvc                                   手動 (トリガー開始)           停止
NlaSvc                                   手動                          停止
nsi                                      自動                          実行
p2pimsvc                                 手動                          停止
p2psvc                                   手動                          停止
PcaSvc                                   自動 (遅延開始、トリガー開始) 実行
PeerDistSvc                              手動                          停止
perceptionsimulation                     手動                          停止
PerfHost                                 手動                          停止
PhoneSvc                                 手動 (トリガー開始)           停止
pla                                      手動                          停止
PlugPlay                                 手動                          実行
PNRPAutoReg                              手動                          停止
PNRPsvc                                  手動                          停止
PolicyAgent                              手動 (トリガー開始)           停止
Power                                    自動                          実行
PrintNotify                              手動                          停止
ProfSvc                                  自動                          実行
PushToInstall                            手動 (トリガー開始)           停止
QWAVE                                    手動                          停止
RasAuto                                  手動                          停止
RasMan                                   手動                          実行
RemoteAccess                             無効                          停止
RemoteRegistry                           無効 (トリガー開始)           停止
RetailDemo                               手動                          停止
RmSvc                                    手動                          実行
RpcEptMapper                             自動                          実行
RpcLocator                               手動                          停止
RpcSs                                    自動                          実行
SamSs                                    自動                          実行
SCardSvr                                 手動 (トリガー開始)           停止
ScDeviceEnum                             手動 (トリガー開始)           停止
Schedule                                 自動                          実行
SCPolicySvc                              手動                          停止
SDRSVC                                   手動                          停止
seclogon                                 手動                          停止
SecurityHealthService                    手動                          実行
SEMgrSvc                                 手動 (トリガー開始)           停止
SENS                                     自動                          実行
Sense                                    手動                          停止
SensorDataService                        手動 (トリガー開始)           停止
SensorService                            手動 (トリガー開始)           停止
SensrSvc                                 手動 (トリガー開始)           停止
SessionEnv                               手動                          停止
SgrmBroker                               無効 (トリガー開始)           停止
SharedAccess                             手動 (トリガー開始)           停止
SharedRealitySvc                         手動                          停止
ShellHWDetection                         自動                          実行
shpamsvc                                 無効                          停止
smphost                                  手動                          停止
SmsRouter                                手動 (トリガー開始)           停止
SNMPTrap                                 手動                          停止
spectrum                                 手動 (トリガー開始)           停止
Spooler                                  自動                          実行
sppsvc                                   自動 (遅延開始、トリガー開始) 停止
SSDPSRV                                  手動                          実行
ssh-agent                                無効                          停止
SstpSvc                                  手動                          実行
StateRepository                          自動                          実行
StiSvc                                   手動 (トリガー開始)           停止
StorSvc                                  自動 (遅延開始、トリガー開始) 実行
svsvc                                    手動 (トリガー開始)           停止
swprv                                    手動                          停止
SysMain                                  自動                          実行
SystemEventsBroker                       自動 (トリガー開始)           実行
TapiSrv                                  手動                          停止
TermService                              手動                          停止
TextInputManagementService               自動 (トリガー開始)           実行
Themes                                   自動                          実行
TieringEngineService                     手動                          停止
TimeBrokerSvc                            手動 (トリガー開始)           実行
TokenBroker                              手動                          実行
TrkWks                                   自動                          実行
TroubleshootingSvc                       手動                          停止
TrustedInstaller                         手動                          停止
tzautoupdate                             無効 (トリガー開始)           停止
UevAgentService                          無効                          停止
uhssvc                                   無効                          停止
UmRdpService                             手動                          停止
upnphost                                 手動                          停止
UserManager                              自動 (トリガー開始)           実行
UsoSvc                                   自動 (遅延開始)               実行
VacSvc                                   手動                          停止
VaultSvc                                 手動                          実行
VBoxService                              自動                          実行
vds                                      手動                          停止
vmicguestinterface                       手動 (トリガー開始)           停止
vmicheartbeat                            手動 (トリガー開始)           停止
vmickvpexchange                          手動 (トリガー開始)           停止
vmicrdv                                  手動 (トリガー開始)           停止
vmicshutdown                             手動 (トリガー開始)           停止
vmictimesync                             手動 (トリガー開始)           停止
vmicvmsession                            手動 (トリガー開始)           停止
vmicvss                                  手動 (トリガー開始)           停止
VSS                                      手動                          停止
W32Time                                  手動 (トリガー開始)           停止
WaaSMedicSvc                             手動                          停止
WalletService                            手動                          停止
WarpJITSvc                               手動 (トリガー開始)           停止
wbengine                                 手動                          停止
WbioSrvc                                 手動 (トリガー開始)           停止
Wcmsvc                                   自動 (トリガー開始)           実行
wcncsvc                                  手動                          停止
WdiServiceHost                           手動                          停止
WdiSystemHost                            手動                          実行
WdNisSvc                                 手動                          実行
WebClient                                手動 (トリガー開始)           停止
webthreatdefsvc                          手動 (トリガー開始)           実行
Wecsvc                                   手動                          停止
WEPHOSTSVC                               手動 (トリガー開始)           停止
wercplsupport                            手動                          停止
WerSvc                                   手動 (トリガー開始)           停止
WFDSConMgrSvc                            手動 (トリガー開始)           停止
WiaRpc                                   手動                          停止
WinDefend                                自動                          実行
WinHttpAutoProxySvc                      手動                          実行
Winmgmt                                  自動                          実行
WinRM                                    手動                          停止
wisvc                                    手動 (トリガー開始)           停止
WlanSvc                                  手動                          停止
wlidsvc                                  手動 (トリガー開始)           実行
wlpasvc                                  手動 (トリガー開始)           停止
WManSvc                                  手動                          停止
wmiApSrv                                 手動                          停止
WMPNetworkSvc                            手動                          停止
workfolderssvc                           手動                          停止
WpcMonSvc                                手動                          停止
WPDBusEnum                               手動 (トリガー開始)           停止
WpnService                               自動                          実行
wscsvc                                   自動 (遅延開始)               実行
WSearch                                  自動 (遅延開始)               実行
wuauserv                                 手動 (トリガー開始)           実行
WwanSvc                                  手動                          停止
XblAuthManager                           手動                          停止
XblGameSave                              手動 (トリガー開始)           停止
XboxGipSvc                               手動 (トリガー開始)           停止
XboxNetApiSvc                            手動                          停止
AarSvc                             手動                          停止
BcastDVRUserService                手動                          停止
BluetoothUserService               手動 (トリガー開始)           停止
CaptureService                     手動                          停止
cbdhsvc                            自動 (遅延開始)               実行
CDPUserSvc                         自動                          実行
CloudBackupRestoreSvc              手動                          停止
ConsentUxUserSvc                   手動                          停止
CredentialEnrollmentManagerUserSvc 手動                          停止
DeviceAssociationBrokerSvc         手動                          停止
DevicePickerUserSvc                手動                          停止
DevicesFlowUserSvc                 手動                          停止
MessagingService                   手動 (トリガー開始)           停止
NPSMSvc                            手動                          実行
OneSyncSvc                         自動 (遅延開始)               実行
P9RdrService                       手動 (トリガー開始)           停止
PenService                         手動 (トリガー開始)           停止
PimIndexMaintenanceSvc             手動                          実行
PrintWorkflowUserSvc               手動 (トリガー開始)           停止
UdkUserSvc                         手動                          実行
UnistoreSvc                        手動                          実行
UserDataSvc                        手動                          実行
webthreatdefusersvc                自動                          実行
WpnUserService                     自動                          実行
```
</details>

<details>
<summary>Tiny11</summary>

```
サービス名                               スタートアップの種類          状態
----------                               --------------------          ----
AJRouter                                 手動 (トリガー開始)           停止
ALG                                      手動                          停止
AppIDSvc                                 手動 (トリガー開始)           実行
Appinfo                                  手動 (トリガー開始)           実行
AppMgmt                                  手動                          停止
AppReadiness                             手動                          停止
AppVClient                               無効                          停止
AppXSvc                                  手動 (トリガー開始)           実行
AssignedAccessManagerSvc                 手動 (トリガー開始)           停止
AudioEndpointBuilder                     自動                          実行
Audiosrv                                 自動                          実行
autotimesvc                              手動 (トリガー開始)           停止
AxInstSV                                 手動                          停止
BDESVC                                   手動 (トリガー開始)           停止
BFE                                      自動                          実行
BITS                                     手動                          停止
BrokerInfrastructure                     自動                          実行
BTAGService                              手動 (トリガー開始)           停止
BthAvctpSvc                              手動 (トリガー開始)           停止
bthserv                                  手動 (トリガー開始)           停止
camsvc                                   手動 (トリガー開始)           実行
CDPSvc                                   自動 (遅延開始、トリガー開始) 実行
CertPropSvc                              手動 (トリガー開始)           停止
ClipSVC                                  手動 (トリガー開始)           停止
cloudidsvc                               手動                          停止
COMSysApp                                手動                          停止
CoreMessagingRegistrar                   自動                          実行
CryptSvc                                 自動 (トリガー開始)           実行
CscService                               手動 (トリガー開始)           停止
DcomLaunch                               自動                          実行
dcsvc                                    手動 (トリガー開始)           停止
defragsvc                                手動                          停止
DeviceAssociationService                 手動 (トリガー開始)           停止
DeviceInstall                            手動 (トリガー開始)           停止
DevQueryBroker                           手動 (トリガー開始)           停止
Dhcp                                     自動                          実行
diagnosticshub.standardcollector.service 手動                          停止
diagsvc                                  手動 (トリガー開始)           停止
DiagTrack                                自動                          実行
DialogBlockingService                    無効                          停止
DispBrokerDesktopSvc                     自動                          実行
DisplayEnhancementService                手動 (トリガー開始)           停止
DmEnrollmentSvc                          手動                          停止
dmwappushservice                         手動 (トリガー開始)           停止
Dnscache                                 自動 (トリガー開始)           実行
DoSvc                                    自動 (遅延開始、トリガー開始) 実行
dot3svc                                  手動                          停止
DPS                                      自動                          実行
DsmSvc                                   手動 (トリガー開始)           停止
DsSvc                                    手動 (トリガー開始)           停止
DusmSvc                                  自動                          実行
EapHost                                  手動                          停止
edgeupdate                               手動                          停止
edgeupdatem                              手動                          停止
EFS                                      手動 (トリガー開始)           停止
embeddedmode                             手動 (トリガー開始)           停止
EntAppSvc                                手動                          停止
EventLog                                 自動                          実行
EventSystem                              自動                          実行
fdPHost                                  手動                          停止
FDResPub                                 手動 (トリガー開始)           停止
fhsvc                                    手動 (トリガー開始)           停止
FontCache                                自動                          実行
FrameServer                              手動 (トリガー開始)           停止
FrameServerMonitor                       手動 (トリガー開始)           停止
GameInputSvc                             手動 (トリガー開始)           停止
gpsvc                                    自動 (トリガー開始)           実行
GraphicsPerfSvc                          手動 (トリガー開始)           停止
hidserv                                  手動 (トリガー開始)           停止
hns                                      手動 (トリガー開始)           実行
HvHost                                   手動 (トリガー開始)           停止
icssvc                                   手動 (トリガー開始)           停止
IKEEXT                                   手動 (トリガー開始)           停止
InstallService                           手動                          停止
InventorySvc                             手動                          停止
iphlpsvc                                 自動                          実行
IpxlatCfgSvc                             手動 (トリガー開始)           停止
KeyIso                                   手動 (トリガー開始)           実行
KtmRm                                    手動 (トリガー開始)           停止
LanmanServer                             自動 (トリガー開始)           実行
LanmanWorkstation                        自動                          実行
lfsvc                                    手動 (トリガー開始)           停止
LicenseManager                           手動 (トリガー開始)           実行
lltdsvc                                  手動                          停止
lmhosts                                  手動 (トリガー開始)           実行
LSM                                      不明                          実行
LxpSvc                                   手動                          停止
LxssManager                              手動                          停止
MapsBroker                               不明                          停止
McpManagementService                     不明                          停止
MixedRealityOpenXRSvc                    手動                          停止
mpssvc                                   自動                          実行
MSDTC                                    手動                          停止
MSiSCSI                                  手動                          停止
msiserver                                手動                          停止
MsKeyboardFilter                         無効                          停止
NaturalAuthentication                    手動 (トリガー開始)           停止
NcaSvc                                   手動 (トリガー開始)           停止
NcbService                               手動 (トリガー開始)           実行
NcdAutoSetup                             手動 (トリガー開始)           停止
Netlogon                                 手動                          停止
Netman                                   手動                          停止
netprofm                                 手動                          実行
NetSetupSvc                              不明 (トリガー開始)           停止
NetTcpPortSharing                        無効                          停止
NgcCtnrSvc                               手動 (トリガー開始)           停止
NgcSvc                                   手動 (トリガー開始)           停止
NlaSvc                                   手動                          停止
nsi                                      自動                          実行
nvagent                                  手動                          実行
p2pimsvc                                 手動                          停止
p2psvc                                   手動                          停止
PcaSvc                                   自動 (遅延開始、トリガー開始) 実行
perceptionsimulation                     手動                          停止
PerfHost                                 手動                          停止
PhoneSvc                                 手動 (トリガー開始)           停止
pla                                      手動                          停止
PlugPlay                                 手動                          実行
PNRPAutoReg                              手動                          停止
PNRPsvc                                  手動                          停止
PolicyAgent                              手動 (トリガー開始)           停止
Power                                    自動                          実行
PrintNotify                              手動                          停止
ProfSvc                                  自動                          実行
PushToInstall                            手動 (トリガー開始)           停止
QWAVE                                    手動                          停止
RasAuto                                  手動                          停止
RasMan                                   手動                          実行
RemoteAccess                             無効                          停止
RemoteRegistry                           無効 (トリガー開始)           停止
RetailDemo                               手動                          停止
RmSvc                                    手動                          実行
RpcEptMapper                             自動                          実行
RpcLocator                               手動                          停止
RpcSs                                    自動                          実行
SamSs                                    自動                          実行
SCardSvr                                 手動 (トリガー開始)           停止
ScDeviceEnum                             手動 (トリガー開始)           停止
Schedule                                 自動                          実行
SCPolicySvc                              手動                          停止
SDRSVC                                   手動                          停止
seclogon                                 手動                          停止
SecurityHealthService                    手動                          実行
SEMgrSvc                                 手動 (トリガー開始)           停止
SENS                                     自動                          実行
Sense                                    手動                          停止
SensorDataService                        手動 (トリガー開始)           停止
SensorService                            手動 (トリガー開始)           停止
SensrSvc                                 手動 (トリガー開始)           停止
SessionEnv                               手動                          停止
SgrmBroker                               無効 (トリガー開始)           停止
SharedAccess                             手動 (トリガー開始)           実行
SharedRealitySvc                         手動                          停止
ShellHWDetection                         自動                          実行
shpamsvc                                 無効                          停止
smphost                                  手動                          停止
SmsRouter                                手動 (トリガー開始)           停止
SNMPTrap                                 手動                          停止
spectrum                                 手動 (トリガー開始)           停止
Spooler                                  自動                          実行
sppsvc                                   自動 (遅延開始、トリガー開始) 停止
SSDPSRV                                  手動                          停止
ssh-agent                                無効                          停止
SstpSvc                                  手動                          実行
StateRepository                          自動                          実行
StiSvc                                   手動 (トリガー開始)           停止
StorSvc                                  自動 (遅延開始、トリガー開始) 実行
svsvc                                    手動 (トリガー開始)           停止
swprv                                    手動                          停止
SysMain                                  自動                          実行
SystemEventsBroker                       自動 (トリガー開始)           実行
TapiSrv                                  手動                          停止
TermService                              手動                          停止
TextInputManagementService               自動 (トリガー開始)           実行
Themes                                   自動                          実行
TieringEngineService                     手動                          停止
TimeBrokerSvc                            手動 (トリガー開始)           実行
TokenBroker                              手動                          実行
TrkWks                                   自動                          実行
TroubleshootingSvc                       手動                          停止
TrustedInstaller                         手動                          停止
tzautoupdate                             無効 (トリガー開始)           停止
UevAgentService                          無効                          停止
uhssvc                                   無効                          停止
UmRdpService                             手動                          停止
upnphost                                 手動                          停止
UserManager                              自動 (トリガー開始)           実行
UsoSvc                                   自動 (遅延開始)               実行
VacSvc                                   手動                          停止
VaultSvc                                 手動                          実行
VBoxService                              自動                          実行
vds                                      手動                          停止
vmcompute                                手動 (トリガー開始)           停止
vmicguestinterface                       手動 (トリガー開始)           停止
vmicheartbeat                            手動 (トリガー開始)           停止
vmickvpexchange                          手動 (トリガー開始)           停止
vmicrdv                                  手動 (トリガー開始)           停止
vmicshutdown                             手動 (トリガー開始)           停止
vmictimesync                             手動 (トリガー開始)           停止
vmicvmsession                            手動 (トリガー開始)           停止
vmicvss                                  手動 (トリガー開始)           停止
VSS                                      手動                          停止
W32Time                                  手動 (トリガー開始)           停止
WaaSMedicSvc                             不明                          停止
WalletService                            手動                          停止
WarpJITSvc                               手動 (トリガー開始)           停止
wbengine                                 手動                          停止
WbioSrvc                                 手動 (トリガー開始)           停止
Wcmsvc                                   自動 (トリガー開始)           実行
wcncsvc                                  手動                          停止
WdiServiceHost                           手動                          停止
WdiSystemHost                            手動                          停止
WdNisSvc                                 手動                          実行
WebClient                                手動 (トリガー開始)           停止
webthreatdefsvc                          手動 (トリガー開始)           実行
Wecsvc                                   手動                          停止
WEPHOSTSVC                               手動 (トリガー開始)           停止
wercplsupport                            手動                          停止
WerSvc                                   手動 (トリガー開始)           停止
WFDSConMgrSvc                            手動 (トリガー開始)           停止
WiaRpc                                   手動                          停止
WinDefend                                自動                          実行
WinHttpAutoProxySvc                      手動                          実行
Winmgmt                                  自動                          実行
WinRM                                    手動                          停止
wisvc                                    手動 (トリガー開始)           停止
WlanSvc                                  手動                          停止
wlidsvc                                  手動 (トリガー開始)           停止
wlpasvc                                  手動 (トリガー開始)           停止
WManSvc                                  手動                          停止
wmiApSrv                                 手動                          停止
WMPNetworkSvc                            手動                          停止
workfolderssvc                           手動                          停止
WpcMonSvc                                手動                          停止
WPDBusEnum                               手動 (トリガー開始)           停止
WpnService                               自動                          実行
wscsvc                                   自動 (遅延開始)               実行
WSearch                                  自動 (遅延開始)               実行
WslInstaller                             自動                          停止
WSLService                               自動                          実行
wuauserv                                 手動 (トリガー開始)           実行
WwanSvc                                  手動                          停止
XblAuthManager                           手動                          停止
XblGameSave                              手動 (トリガー開始)           停止
XboxGipSvc                               手動 (トリガー開始)           停止
XboxNetApiSvc                            手動                          停止
AarSvc                             手動                          停止
BcastDVRUserService                手動                          停止
BluetoothUserService               手動 (トリガー開始)           停止
CaptureService                     手動                          停止
cbdhsvc                            自動 (遅延開始)               実行
CDPUserSvc                         自動                          実行
CloudBackupRestoreSvc              手動                          停止
ConsentUxUserSvc                   手動                          停止
CredentialEnrollmentManagerUserSvc 手動                          停止
DeviceAssociationBrokerSvc         手動                          停止
DevicePickerUserSvc                手動                          停止
DevicesFlowUserSvc                 手動                          停止
MessagingService                   手動 (トリガー開始)           停止
NPSMSvc                            手動                          実行
OneSyncSvc                         自動 (遅延開始)               実行
P9RdrService                       手動 (トリガー開始)           停止
PenService                         手動 (トリガー開始)           停止
PimIndexMaintenanceSvc             手動                          停止
PrintWorkflowUserSvc               手動 (トリガー開始)           停止
UdkUserSvc                         手動                          実行
UnistoreSvc                        手動                          停止
UserDataSvc                        手動                          停止
webthreatdefusersvc                自動                          実行
WpnUserService                     自動                          実行
```
</details>

## diff

```
$ diff win11.txt tiny11.txt
8c8
&lt; AppReadiness                             手動                          実行
---
> AppReadiness                             手動                          停止
26c26
&lt; ClipSVC                                  手動 (トリガー開始)           実行
---
> ClipSVC                                  手動 (トリガー開始)           停止
44c44
&lt; DisplayEnhancementService                手動 (トリガー開始)           実行
---
> DisplayEnhancementService                手動 (トリガー開始)           停止
55,56c55,56
&lt; edgeupdate                               自動 (遅延開始、トリガー開始) 停止
&lt; edgeupdatem                              手動 (トリガー開始)           停止
---
> edgeupdate                               手動                          停止
> edgeupdatem                              手動                          停止
71a72
> hns                                      手動 (トリガー開始)           実行
75c76
&lt; InstallService                           手動                          実行
---
> InstallService                           手動                          停止
83c84
&lt; lfsvc                                    手動 (トリガー開始)           実行
---
> lfsvc                                    手動 (トリガー開始)           停止
89,91c90,92
&lt; MapsBroker                               自動 (遅延開始)               停止
&lt; McpManagementService                     手動                          停止
&lt; MicrosoftEdgeElevationService            手動                          停止
---
> LxssManager                              手動                          停止
> MapsBroker                               不明                          停止
> McpManagementService                     不明                          停止
105c106
&lt; NetSetupSvc                              不明 (トリガー開始)           実行
---
> NetSetupSvc                              不明 (トリガー開始)           停止
110a112
> nvagent                                  手動                          実行
114d115
&lt; PeerDistSvc                              手動                          停止
153c154
&lt; SharedAccess                             手動 (トリガー開始)           停止
---
> SharedAccess                             手動 (トリガー開始)           実行
163c164
&lt; SSDPSRV                                  手動                          実行
---
> SSDPSRV                                  手動                          停止
193a195
> vmcompute                                手動 (トリガー開始)           停止
204c206
&lt; WaaSMedicSvc                             手動                          停止
---
> WaaSMedicSvc                             不明                          停止
212c214
&lt; WdiSystemHost                            手動                          実行
---
> WdiSystemHost                            手動                          停止
228c230
&lt; wlidsvc                                  手動 (トリガー開始)           実行
---
> wlidsvc                                  手動 (トリガー開始)           停止
238a241,242
> WslInstaller                             自動                          停止
> WSLService                               自動                          実行
262c266
&lt; PimIndexMaintenanceSvc             手動                          実行
---
> PimIndexMaintenanceSvc             手動                          停止
265,266c269,270
&lt; UnistoreSvc                        手動                          実行
&lt; UserDataSvc                        手動                          実行
---
> UnistoreSvc                        手動                          停止
> UserDataSvc                        手動                          停止
```

## 内容確認

[システム サービスの構成に関するガイダンス - Windows IoT Enterprise | Microsoft Learn](https://learn.microsoft.com/ja-jp/windows/iot/iot-enterprise/optimize/services)

純正では実行されているがTiny11では停止しているものを確認する。

- AppReadiness
  - MSStore関連。
- ClipSVC
  - MSStore関連。
- DisplayEnhancementService
  - 輝度調整とかっぽい。
- InstallService
  - MSStoreインストールサービスらしい。止めてもいいらしい。
- lfsvc
  - 位置情報サービスらしい。
- NetSetupSvc
  - ネットワークドライバのインストールの管理らしい。
- SSDPSRV
  - SSDP探索系らしい。
- WdiSystemHost
  - 診断するやつらしい。
- wlidsvc
  - MSアカウントでログインできるようにするやつらしい。
- PimIndexMaintenanceSvc
  - 連絡先データのインデックスを作成するやつっぽい
- UnistoreSvc, UserDataSvc
  - ユーザーデータのインデックスするやつ関連っぽい

## 〆

- MSStore関連は……一応止めないでおくか……
- DisplayEnhancementServiceはノパソにいれるなら必要だな
- PimIndexMaintenanceSvc, UnistoreSvc, UserDataSvcは止めても良いかもね
- WdiSystemHostは止めてみた
- なんか普通にWindows11でいい気がしてきたぞ。

## おまけ

- なんか放置してたら同じくらいのメモリ消費量になっちゃったぞ
- Tiny11、日本語フォントに游ゴシックとメイリオが入ってないのかも。MSゴシックとMS UI Gothicにフォールバックしてる、

![](img/スクリーンショット-2024-04-07-1.10.32-1024x640.png)