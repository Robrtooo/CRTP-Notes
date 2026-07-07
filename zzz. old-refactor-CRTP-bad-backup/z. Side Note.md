### Tools mapping
```powershell
Get-ChildItem C:\AD\Tools -Recurse -File -Include *.exe,*.ps1,*.psm1,*.bat |
 ? FullName -notmatch 'BloodHound-win32-x64|neo4j-community|john-1.9|HeidiSQL|openssl' |
 select -Expand FullName |
 Out-File -Encoding utf8 C:\AD\Tools\tools-paths.txt
notepad C:\AD\Tools\tools-paths.txt
```