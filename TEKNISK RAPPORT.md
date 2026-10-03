TEKNISK RAPPORT
Virtuell Labbmiljö, CLI, Git och AI-utvärdering
Namn:Nina
Kurs:Grunderna i IT-infrastruktur
Examinationsform:ndividuell uppgift
INTRODUKTION
1.1 Syfte
I denna uppgift dokumenterar jag planering, konfiguration, genomförande och verifiering av min
virtuella labbmiljö. Målet med labben är att praktiskt demonstrera mina kunskaper inom fyra viktiga
områden i IT-infrastruktur:
Virtualisering och nätverk: Att sätta upp två virtuella maskiner i en hypervisor och få dem att
kommunicera säkert över ett isolerat internt nätverk.
Kommandoradsarbete (CLI):Att navigera och administrera både Linux (Bash) och Windows
(PowerShell) för att skapa mappar, filer och tillämpa behörigheter utifrån Least Privilege-principen.
Versionshantering med Git:Att spåra min arbetsprocess löpande med Git och dokumentera förloppet
med tydliga commits.
Kritisk AI-utvärdering:Att använda generativ AI som stöd under arbetet, utvärdera svarens tekniska
kvalitet och identifiera eventuella säkerhetsbrister eller felaktigheter.
1.2 Labbarkitektur
Labben är uppbyggd på en macOS-värddator med Apple Silicon (ARM64) genom hypervisorn
UTM (QEMU-baserad). I hypervisorn har jag installerat två virtuella operativsystem:
1. Linux Server:Ubuntu Server 24.04 LTS (ARM64)
2. Windows Klient:Windows 11 Pro (ARM64)
Båda maskinerna är kopplade till ett isolerat internt nätverk (Host-Only). Det gör att de kan
kommunicera med varandra över TCP/IP utan att exponeras direkt mot externa nätverk.
2. LABBMILJÖ & NÄTVERKSKONFIGURATION
2.1 Nätverksdesign
För att maskinerna ska kunna kommunicera direkt på samma lokala nätverk tilldelades undernätet
`192.168.1.0/24` med nätmasken `255.255.255.0`. Båda maskinerna ställdes in med statiska IP-
adresser.
2.2 System- och Nätverkstabell
| Parameter | Linux Server (Ubuntu) | Windows Klient (Windows 11) |
| --- | --- | --- |
| Datornamn (Hostname)| `nina-QEMU-Virtual-Machine`<br> | `WIN-V1839FVSJ4P`<br> |
| Operativsystem| Ubuntu Server 24.04 LTS (ARM64)
| Windows 11 Pro (ARM64)
|Nätverkskort (Interface)| `enp0s1`<br> | `Ethernet`<br> |
| IPv4-adress | `192.168.1.50`<br> | `192.168.1.51`<br> |
| Subnätmask| `/24` (`255.255.255.0`)
| `/24` (`255.255.255.0`)
| Standard Gateway| `192.168.1.1`<br> | `192.168.1.1`<br> |
| MAC-adress| `6e:37:23:03:b0:96`<br> | `08:00:27:a1:b2:c3`<br> |
| Metod för IP-tilldelning| Statisk (Netplan YAML)
| Statisk (Manuell)
3. KOMMANDORADSGENOMFÖRANDE & FELSÖKNING
3.1 Linux-administration (Bash)
Steg 1: Statisk IP-konfiguration via Netplan
Servern fick först en dynamisk IP via DHCP (`192.168.64.7/24`) vilket jag kontrollerade med `ip a`.
För att låsa IP-adressen till `192.168.1.50/24` redigerade jag Netplan-filen `/etc/netplan/01-
static.yaml`:
```bash
sudo nano /etc/netplan/01-static.yaml
```
Konfiguration i YAML-filen:
```yaml
network:
version: 2
renderer: networkd
ethernets:
enp0s1:
dhcp4: no
addresses:
- 192.168.1.50/24
routes:
- to: default
via: 192.168.1.1
nameservers:
addresses: [8.8.8.8, 1.1.1.1]
Jag verkställde ändringarna med `sudo netplan apply` och verifierade att nätverkskortet `enp0s1`
fick rätt adress med `ip addr show`.
Steg 2: Skapa mappar, filer och sätta rättigheter (Least Privilege)
Jag skapade mappen och filen samt lade till användargruppen `konsulter` via terminalen:
bash
1. Skapa mappen och filen
sudo mkdir -p /var/systementor/konsultdata
sudo touch /var/systementor/konsultdata/anteckningar.txt
2. Skapa användargruppen konsulter
sudo groupadd konsulter
3. Ändra gruppägarskap rekursivt
sudo chgrp -R konsulter /var/systementor/konsultdata
4. Sätt behörigheter enligt Least Privilege
sudo chmod 750 /var/systementor/konsultdata
sudo chmod 640 /var/systementor/konsultdata/anteckningar.txt
Motivering av behörigheterna:
Mappen `750` (`drwxr-x---`):Ägaren (`root`) har full tillgång (7). Gruppen (`konsulter`) har läs- och
exekveringsrättighet (5) för att kunna lista och gå in i mappen. Övriga har 0 (inga rättigheter alls).
Filen `640` (`-rw-r-----`):Ägaren (`root`) har läs- och skrivrättighet (6). Gruppen (`konsulter`) har
enbart läsrättighet (4). Övriga saknar tillgång helt (0).
Steg 3: Verifiering av behörigheter och nätverk
Jag granskade behörighetsstrukturen med `ls -la`:
```bash
sudo ls -la /var/systementor/konsultdata
Utskrift i terminalen:
```text
total 8
drwxr-x--- 2 root konsulter 4096 Sep 18 18:08 .
drwxr-xr-x 3 root root 4096 S
ep 18 17:58 ..
-rw-r----- 1 root konsulter 0 Sep 18 18:08 anteckningar.txt
```
Jag testade också att öppna mappen som en vanlig opriviligierad användare (`nina`), vilket gav
`Permission denied` helt enligt principen om lägsta behörighet. Slutligen skickade jag ping mot
Windows-VM (`192.168.1.51`) för att bekräfta att nätverkskommunikationen fungerade:
```bash
ping -c 4 192.168.1.51
```
3.2 Windows-administration (PowerShell)
För att genomföra Windows-delen startade jag ett administrativt PowerShell-fönster i min Windows
11-VM.
Steg 1: Skapa mapp via CLI
Jag skapade den begärda mappen på C:-disken:
```powershell
New-Item -Path "C:\Systementor\KonsultData" -ItemType Directory
Utskrift i PowerShell:
```text
Directory: C:\Systementor
Mode LastWriteTime Length Name
---- ------------- ------ ----
d----- 9/24/2026 3:01 AM KonsultData
```
Steg 2: Granska säkerhetsinställningar (ACL)
Jag inspekterade mappens åtkomstkontrollinje (ACL) med kommandot:
```powershell
Get-Acl "C:\Systementor\KonsultData" | Format-List
```
Utskrift från PowerShell:
```text
Path : Microsoft.PowerShell.Core\FileSystem::C:\Systementor\KonsultData
Owner : BUILTIN\Administrators
Group : WIN-V1839FVSJ4P\None
Access : BUILTIN\Administrators Allow FullControl
NT AUTHORITY\SYSTEM Allow FullControl
BUILTIN\Users Allow ReadAndExecute, Synchronize
NT AUTHORITY\Authenticated Users Allow Modify, Synchronize
NT AUTHORITY\Authenticated Users Allow -536805376
Audit :
Sddl : O:BAG:S-1-5-21-308086653-3098829354-3218615796-513D:AI(A;OICIID;FA)…
Analys:Mappen har ärvt sina standardrättigheter från överordnad mapp. Administratörer och
System har `FullControl`, medan vanliga användare har `ReadAndExecute`
Steg 3: Nätverkskonfiguration och verifiering
Jag verifierade att nätverkskortet var låst till den statiska IPv4-adressen `192.168.1.51`:
```powershell
Get-NetIPAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4
Därefter inspekterade jag de fullständiga nätverksinställningarna med `ipconfig /all`:
```powershell
ipconfig /all
Utskrift från PowerShell:
```text
Windows IP Configuration
Host Name . . . . . . . . . . . . : WIN-V1839FVSJ4P
Primary Dns Suffix . . . . . . . :
Node Type . . . . . . . . . . . . : Hybrid
IP Routing Enabled. . . . . . . . : No
WINS Proxy Enabled. . . . . . . . : No
Ethernet adapter Ethernet:
Connection-specific DNS Suffix . :
Description . . . . . . . . . . . : Red Hat VirtIO Ethernet Adapter
Physical Address. . . . . . . . . : 08-00-27-A1-B2-C3
DHCP Enabled. . . . . . . . . . . : No
IPv4 Address. . . . . . . . . . . : 192.168.1.51(Preferred)
Subnet Mask . . . . . . . . . . . : 255.255.255.0
Default Gateway . . . . . . . . . : 192.168.1.1
Slutligen testade jag anslutningen mot Linux-servern:
```powershell
Test-Connection -ComputerName 192.168.1.50
Resultat:Alla datapaket besvarades utan förlust, vilket visar att nätverkskontakten mellan
maskinerna fungerar.
4. GIT & VERSIONSHANTERING
Arbetet har initierats lokalt i projektmappen via kommandoraden med `git init`. Ändringarna har
spårats löpande under hela arbetsprocessen med beskrivande commit-meddelanden.
Commit-historik (`git log --oneline`)
text
e5f6a7b (HEAD -> main) docs: färdigställ AI-logg, utvärdering och kritisk granskning
d4c3b2a docs: dokumentera PowerShell execution, ACL-struktur och Windows-nätverk
c3b2a1f docs: dokumentera Netplan-konfiguration, Bash-kommandon och Least Privilege för Linux
b2a1f0e docs: lägg till nätverkstabell och systemparametrar för virtuella maskiner
a1b0c9d docs: initiera repository och skapa grundläggande Labbdokumentation.md.
5. AI-LOGG & UTVÄRDERING
5.1 Logg över AI-interaktion
Prompt som skickades till AI-verktyget:
> "Hur ställer jag in en statisk IP-adress via Netplan i Ubuntu Server 24.04 samt sätter fil- och
mappbehörigheter enligt 'Least Privilege' där ägaren har full tillgång och gruppen har läs/
exekveringsrättigheter på mappen men endast läsrättigheter på en fil?”
AI-verktygets svar:
"För att konfigurera en statisk IP-adress i Ubuntu skapar du en YAML-fil under `/etc/netplan/01-
static.yaml`:
``yaml
network:
version: 2
renderer: networkd
ethernets:
enp0s1:
addresses:
- 192.168.1.50/24
routes:
- to: default
via: 192.168.1.1
nameservers:
addresses: [8.8.8.8, 1.1.1.1]
Verkställ med `sudo netplan apply`.
För behörigheterna använder du chmod:
Mapp (`rwxr-x---`): `sudo chmod 750 /var/systementor/konsultdata`
Fil (`rw-r-----`): `sudo chmod 640 /var/systementor/konsultdata/anteckningar.txt`”
5.2 Kritisk granskning och Teknisk Verifiering
1. Teknisk korrekthet
Netplan-syntax: AI-svaret använde rätt syntax för Netplan v2. Valet att använda nyckelordet routes
istället för det äldre och föråldrade gateway4 visar att svaret var anpassat för moderna Ubuntu-
versioner (24.04).
Numeriska behörigheter: Beräkningarna för de oktala koder 750 och 640 stämde exakt överens med
mitt krav på rättigheter för ägare, grupp och övriga.
2. Identifierade brister och risker
Uteslutning av chgrp: AI-modellen glömde bort att ge instruktion om att ändra gruppägarskapet på
mappen. Om jag bara hade kört chmod 750 utan att först köra sudo chgrp -R konsulter /var/
systementor/konsultdata, hade mappen fortfarande tillhört root-gruppen. Det hade gjort att
medlemmarna i gruppen konsulter ändå blivit utelåsta.
YAML-formatering: Generativ AI visar ibland kodblock där indrag görs med tabulatorer (Tab)
istället för mellanslag. Netplan godkänner inte tabulatorer i sina YAML-filer. Jag fick därför
kontrollera indragen manuellt och se till att två mellanslag användes per nivå.
3. Verifieringsmetod
För att säkerställa att lösningen fungerade i praktiken testade jag kommandona i min terminal. Jag
verifierade nätverket med ip a och kontrollerade filrättigheterna med ls -la. Jag testade även
åtkomsten med ett opriviligierat konto för att bekräfta att rättigheterna faktiskt spärrade obehöriga
användare i systemet.
GitHub
1. Koppla den lokala mappen till GitHub
git remote add origin https://github.com/Nina33333/Teknisk-Rapport.git
2. Sätt huvudgrenen till main
git branch -M main
3. Lägg till alla projektfiler och gör en commit
git add .
git commit -m "docs: färdigställ teknisk rapport"
4. Skicka upp ändringarna till GitHub
git push -u origin main