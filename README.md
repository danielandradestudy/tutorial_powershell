## 🖥️ Alguns comandos de powershelll
Dicas rápidas de Powershell, inspirado no curso do Professor Ramos! Sempre que eu estiver aprendendo algo, irei postar por aqui 🤖 


	[Link do curso completo](https://www.youtube.com/playlist?list=PL35Zp8zig6slB_EaLbwKP57L9weBfICtS)https://www.youtube.com/playlist?list=PL35Zp8zig6slB_EaLbwKP57L9weBfICtS)
# O basico inicial, a sigla (OU) significa unidades organizacionais!  

ni arq.txt # criar um arquivo   
rni arq.txt novo.txt # renomear um arquivo  
cp arq.txt c:\ # copiar arq para raiz  
mv # mover arq  
erase arq.txt # apagar  
erase -force -Recurse pasta #apagar arq ou pasta na ignorância   

# Fazer backup dos drivers externos do computador, ou seja não inclui os drivers instalados pelo windows update!

Export-WindowsDriver -Online -Destination C:\Drivers

# Ver uma lista dos drivers exportados 

$BackupDrivers = Export-WindowsDriver -Online -Destination C:\Drivers
$BackupDrivers | Select-Object ClassName, ProviderName, Date, Version | Export-CSV C:\Drivers\lista.txt

# Exibir algumas infos sobre o seu computador e também sobre a BIOS

Get-WmiObject -Class Win32_ComputerSystem
Get-WmiObject -Class Win32_BIOS -ComputerName .

# Listar os 5 processos comedores de RAM

ps | sort –p ws | select –last 5


# Adcionar programas e recursos:  

Import-Module ServerMasnager # importar o modulo responsavel para mecher nos próximos comandos  
get-windowsFeature *dns* # ver todos os servicos relacionados a dns  
add-windowsfeature -name windows-server-backup -restart # adiciona recurso de backup  
install-windowsfeature -name rsat-fax -restart  
remove-windows-feature -name rsat-fsrm-mgmt -restart # remover servico  

# Gerenciar servicos  

get-service # ver todos os serviços     
get-service | where-object { $_.Status -eq "running" } # ver os serviços em exec  
get-service | where-object { $_.Status -eq "stopped" } # ver os serviços parados  
get-service dhcp # saber algo sobre o servico  
get-service *dhcp*  # saber tudo que está relacionado a dhcp  
stop-service spooler # parar serviço  
start-service spooler  # startar um serviço  
restart-service wuauserv # reiniciar um serviço  

# Visualizador de eventos  

Get-EventLog (qual?):  

Get-EventLog Application   
Get-EventLog Security  
Get-EventLog System  
Get-EventLog -Newest 8 Security  
Get-EventLog -Newest 10 Application  
Get-EventLog Security | where-object { $_.InstanceID -eq 4660} = somente eventos relacionados ao id  
Get-EventLog System | where-object { $_.EntryType -eq "Error" }  
Get-EventLog System | where-object { $_.EntryType -eq "Warning" }  

Gerenciamento de rede:  

Get-NetIPAddress  
ncpa.cpl # atalho para GUI de Painel de Controle\Rede e Internet\Conexões de Rede  
Get-NetIPConfiguration - ver o meu ip  

# Adicionar Ip e DNS de forma estática  
New-NetIPAddress (IP) -InterfaceAlias Ethernet -DefaultGateway (GATEWAY) -AddressFamily ipv4 -PrefixLength 8  
Set-DnsClientServerAddress -InterfaceAlias ethernet -ServerAddresses 8.8.4.4,8.8.8.8  


# Deixar tudo via DHCP  
Set-NetIPInterface -InterfaceAlias ethernet -Dhcp Enabled  
Set-DnsClientServerAddress -InterfaceAlias ethernet -ResetServerAddresses  
Remove-NetRoute -InterfaceAlias ethernet  
Restart-NetAdapter -InterfaceAlias ethernet  



# Mechendo com ActiveDirectory   

get-service *adws  
Get-Module -ListAvailable # ver todos os modulos instalados   
import-module activedirectory  
get-comand -Module ActiveDirectory # ver os comandos disponiveis para usar  
get-childitem ad: # ver informações do meu doninio ou também pode usar --> diir ad:  
get-childitem ad: (tab para fitrar)  
get-childitem (tab) > dominio.txt #redirecionar a saida para um arquivo texto  
get-adforest #pegar infos da floresta (>>) add info sem sobrescrever  
get-addomain
get-addefaultdomainpasswordpolicy   
get-adcomputer * # ver todos os computadopres que fazem parte do dominio  
get-adcomputer  -filter * > micros.txt  
get-adcomputer  -filter * | convertto-html | out-file "micros.html"  
get-addomaincontroler # ver infos do controlador de dominio  
get-adorganizationalunit -filter *  
get-aduser -filter * | more # ver os usuários   
get-aduser (user)  
get-aduser (user) -properties  
get-aduser lais -properties * get-aduser (user) out-gridview  
get-aduser lais -propertied lastlogondate # ultima vez que o user fez logon  
get-aduser =filter *  | where {$_.enabled -eq $false} # ver todos os usrers desabiltados  
Get-ADOrganizationalUnit -Filter * # ver quem faz parte da sua Unidades organizacionais "OU"  
New-ADOrganizationalUnit -Name "professores" -Path "dc=aulaead,dc=dc" # adicionar a OU professores no dominio auleead.dc  
New-ADOrganizationalUnit -Name "TI" -Path "ou=professores,dc=aulaead,dc=dc" # criar uma sub ou dentro de profesores   

# Criar a ou administração, renomear para ADM e mover para professores  

New-ADOrganizationalUnit -Name "administração" -Path "dc=aulaead,dc=dc"  
Set-ADOrganizationalUnit -Identity "ou=administracao,dc=aulaead,dc=dc" -ProtectedFromAccide  
ntalDeletion $false  
Rename-ADObject "ou=administracao,dc=aulaead,dc=dc" -NewName "ADM"  
Move-ADObject "ou=adm,dc=aulaead,dc=dc" -TargetPath "ou=professores,dc=aulaead,dc=dc"  
Get-ADObject -Filter * -SearchBase "ou=professores,dc=aulaead,dc=dc"  


# Criar a OU teste e remover  

New-ADOrganizationalUnit -Name "TESTE" -Path "dc=aulaead,dc=dc"  
Set-ADOrganizationalUnit -Identity "ou=teste,dc=aulaead,dc=dc" -ProtectedFromAccide  
ntalDeletion $false  
Remove-ADOrganizationalUnit "ou=teste,dc=aulaead,dc=dc" -Recursive  

# Gerenciar usuários    

New-ADUser -SamAccountName "leandro.ramos" -GivenName "Leandro" -Surname "Ramos" -Name "Leandro Ramos" -UserPrincipalName "leandro.ramos@aulaead.dc" -AccountPassword (ConvertTo-SecureString -AsPlainText "123@Mudar" -Force) -ChangePasswordAtLogon $true -Enabled $true -Path "ou=professores,dc=aulaead,dc=dc"  

# ver se o user foi criado e está na OU professores     

Get-ADUser leandro.ramos  
Get-ADUser leandro.ramos -Properties *  
Get-ADObject -Filter * -SearchBase "ou=professores,dc=aulaead,dc=dc"  

# Criar o user rivas e mover para a OU ti   

New-ADUser -SamAccountName "rivas.reis" -GivenName "rivas" -Surname "reis" -Name "Rivas Reis" -UserPrincipalName "rivas.reis@aulaead.dc" -AccountPassword (ConvertTo-SecureString -AsPlainText "123@Mudar" -Force) -ChangePasswordAtLogon $true -Enabled $true -Path "ou=professores,dc=aulaead,dc=dc"  

Get-ADUser "rivas.reis" | Move-ADObject -TargetPath "ou=ti,ou=professores,dc=aulaead,dc=dc"  

# Alterar a senha do rivas  
Set-ADAccountPassword -Identity "rivas.reis" -reset -NewPassword (ConvertTo-SecureString -AsPlainText "Mudar@321" -Force)   

# Desabilitar, habilitar ou remover a conta do rivas    

Enable-ADAccount -Identity "rivas.reis"  
Disable-ADAccount -Identity "rivas.reis"  
Remove-ADUser rivas.reis  

Get-ADUser rivas.reis -Properties *  

# Mudar o departamento de todos os user  
Get-ADUser -Filter * -SearchBase "ou=alunos,dc=aulaead,dc=dc" | Set-ADUser -Department AULAEAD  
