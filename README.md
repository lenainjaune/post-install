# post-install
Scripts à exécuter une fois X installé

## Windows 7
post-install_win7.bat
``` batch
@ECHO OFF

:: Windows 7 post-install script (to be executed elevated)
:: NEEDS : MSI Bonjour


:: Elevation (https://stackoverflow.com/questions/7985755/how-to-detect-if-cmd-is-running-as-administrator-has-elevated-privileges)
AT > NUL
IF %ERRORLEVEL% NEQ 0 ECHO Ce script doit etre execute en tant qu'administrateur ! & GOTO QUIT

:: BONJOUR (ping *.local au lieu de ping IP)
FOR /F %%I IN ('DIR "%~dp0\bonjour*" /B /S 2^> NUL') DO SET "bonjour=%%I"
IF "%bonjour%"=="" ECHO Le service Bonjour n'a pas ete trouve. & ECHO Copier le MSI correspondant dans le dossier de ce script. & GOTO QUIT


ECHO Post installation de Windows 7

START /WAIT msiexec /i %bonjour% /qn
ECHO Service bonjour installe

:: FIREWALL RULE
::  Allow ICMP from LAN (https://www.howtogeek.com/howto/windows-vista/allow-pings-icmp-echo-request-through-your-windows-vista-firewall/)
netsh advfirewall firewall add rule name="ICMP Allow incoming LAN echo request" protocol=icmpv4:8,any dir=in action=allow remoteip=localsubnet > NUL
ECHO Parefeu installe (ping *.local possible)


ECHO Post-installation effectuee avec succes !
GOTO QUIT


:QUIT
ECHO.
ECHO Appuyer sur une touche pour quitter ...
PAUSE > NUL
ECHO ON
EXIT /B
```
