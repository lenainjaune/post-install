# post-install
Scripts à exécuter une fois le système installé

## Windows 7
post-install_win7.bat
``` batch
:: Windows 7 post-install script (to be executed elevated)
:: NEEDS : MSI Bonjour

@ECHO OFF

:: Elevation needed
:: https://stackoverflow.com/questions/7985755/how-to-detect-if-cmd-is-running-as-administrator-has-elevated-privileges
AT > NUL
IF %ERRORLEVEL% NEQ 0 (
 ECHO Ce script doit etre execute en tant qu'administrateur !
 GOTO QUIT
)

:: Save the current Code Page and activate West European CP1252 (included french)
:: In french we need sometimes the West European Code Page for the accentuated characters
:: https://ss64.com/nt/chcp.html
:: ex : "Page de codes active : 850" -> "850"
FOR /F "tokens=*" %%C IN ('CHCP') DO (
 SET "CPF=%%C"
)
SET "CP=%CPF:*: =%"
CHCP 1252 > NUL

:: NEED Bonjour service
FOR /F %%I IN ('DIR "%~dp0\bonjour*" /B /S 2^> NUL') DO (
 SET "bonjour=%%I"
)
IF "%bonjour%"=="" (
 ECHO Le service Bonjour n'a pas ete trouve.
 ECHO Copier le MSI correspondant dans le dossier de ce script.
 GOTO QUIT
)

ECHO Post-installation de Windows 7

START /WAIT msiexec /i "%bonjour%" /qn
ECHO Service Bonjour installe (ping *.local possible) ...

:: Enable access to Terminal Server (RDP)
REG ADD "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server"^
 /v fDenyTSConnections /t REG_DWORD /d 0 /f > NUL
ECHO Bureau a distance actif ...


:: FIREWALL RULES (WITHOUT DISABLE IT)

:: https://www.howtogeek.com/howto/windows-vista/allow-pings-icmp-echo-request-through-your-windows-vista-firewall/
netsh advfirewall firewall delete rule ^
 "ICMP Allow incoming LAN echo request" > NUL
netsh advfirewall firewall add rule ^
 name="ICMP Allow incoming LAN echo request"^
 protocol=icmpv4:8,any dir=in action=allow remoteip=localsubnet > NUL
ECHO Pare-feu autorise ICMP depuis LAN ...

:: "Remote Desktop" is named in french "Bureau à distance", so we need CP 1252
netsh advfirewall firewall set rule^
 group="Bureau à distance" new enable=Yes > NUL
ECHO Pare-feu autorise Bureau a distance ...


ECHO Fin de post-installation !
GOTO QUIT


:QUIT
CHCP %CP% > NUL
ECHO.
ECHO Appuyer sur une touche pour quitter ...
PAUSE > NUL
ECHO ON
EXIT /B```
