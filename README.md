---
description: >-
  Este projeto demonstra a configuração de um laboratório virtual de segurança
  ofensiva (Red Team) utilizando o VirtualBox. O cenário envolve dua VMs:
---

# README.md

1. **Atacante:** Kali Linux (versão 2024.1 ou superior)&#x20;
2. **Alvo:** Metasploitable 2 (uma máquina intencionalmente vulnerável)

## Ataque Brute Force de Senhas com Medusa no Kali Linux.

Este projeto demonstra a configuração de um laboratório virtual de segurança ofensiva (Red Team) utilizando o VirtualBox. O cenário envolve duas máquinas virtuais:

1. **Atacante:** Kali Linux (versão 2024.1 ou superior)
2. **Alvo:** Metasploitable 2 (uma máquina intencionalmente vulnerável)

O objetivo principal é executar um ataque de **Brute Force** contra o serviço FTP do alvo utilizando a ferramenta **Medusa**, um brute forcer paralelo rápido.

### 📋 Tópicos

* Configuração do Laboratório
* Preparação do Kali Linux
* Preparação do Metasploitable 2
* Criação dos Wordlists
* Execução do Ataque com Medusa
* Análise dos Resultados
* Considerações Finais

***

### Configuração do Laboratório.

Para reproduzir este experimento, é necessário ter o VirtualBox instalado. A topologia de rede recomendada é a **NAT Network** ou **Host-Only**, permitindo comunicação direta entre as VMs sem depender da internet externa para o tráfego local.

#### Topologia de Rede

* **Kali Linux:** IP `192.168.56.10` (Adaptador 1 - Host-Only)
* **Metasploitable 2:** IP `192.168.56.11` (Adaptador 1 - Host-Only)

> **Nota:** Utilize a interface de rede "Host-Only" do VirtualBox para criar uma rede virtual privada (`vboxnet0`).

***

### Preparação do Kali Linux.

1. **Download:** Baixe a imagem ISO do Kali Linux (Workstation Edition).
2. **Criação da VM:**
   * **Sistema Operacional:** Linux (Debian 64-bit).
   * **Memória:** 4096 MB (mínimo).
   * **Processador:** 2 vCPUs.
   * **Disco Virtual:** 40 GB (Dinâmico).
3. **Instalação:**
   * Selecione a instalação gráfica (Graphical Install).
   * Configuração de rede: Defina o IP estático `192.168.56.10` na interface `eth0` (ou `enp0s3` dependendo da versão).
   * Usuário padrão: `kali` / Senha: `kali` (padrão de fábrica).
4.  **Atualizações:**

    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install medusa nmap -y
    ```

***

### Preparação do Metasploitable 2.

1. **Download:** Baixe a imagem `.ova` do Metasploitable 2 no site oficial da Rapid7.
2. **Importação:** Importe a máquina no VirtualBox.
3. **Configuração de Rede:**
   * Ajuste a interface de rede para **Host-Only Adapter**.
   * Defina o IP estático `192.168.56.11`.
4. **Inicialização:**
   * Inicie a VM.
   * Login padrão: `msfadmin` / `msfadmin`.
   * O serviço SSH (porta 22) deve estar ativo.

> **Verificação de Conectividade:** A partir do Kali Linux, verifique se o alvo está acessível:
>
> ```bash
> ping 192.168.56.11
> nmap -sV -p 22 192.168.56.11
> ```

***

### Criação dos Wordlists.

Para otimizar o ataque, criaremos dois arquivos de texto contendo os possíveis usuários e senhas.

### Execução do Ataque com o Medusa.

O Medusa é um _parallel network login cracker_ extremamente rápido e silencioso. Ele permite testar milhares de combinações de usuário/senha em paralelo.

#### Cenário 1: Força Bruta em FTP (High-Volume Brute Force)

Este cenário demonstra um ataque exaustivo contra um servidor FTP usando dicionários grandes e múltiplas threads.

#### Cenário 2: Automação de Tentativas em Formulário Web

O Medusa possui um módulo `http-form` que permite atacar formulários de login personalizados (HTML).

#### Cenário 3: _Password Spraying_ em SMB com Enumeração

_Password Spraying_ é uma técnica de força bruta inversa: em vez de testar muitas senhas para um usuário, você testa **uma senha fraca para muitos usuários**. Isso evita bloqueios de conta.

### Análise de resultados.

O Medusa exibirá a saída no terminal. Credenciais válidas são destacadas.

**Exemplo de Saída:**

```
textDownloadCopy codeMedusa v3.12 [192.168.56.10]
...
[+] Host: 192.168.56.11  User: msfadmin  Pass: msfadmin
[+] Host: 192.168.56.11  User: root     Pass: 123456


```

### Considerações Finais

**Segurança de Senhas:** O ataque demonstrou como senhas fracas e comuns são vulneráveis a ataques automatizados. Em produção, utilize senhas fortes, com no mínimo 12 caracteres, combinando maiúsculas, minúsculas, números e símbolos.

#### Proteção contra Força Bruta em FTP

O objetivo é impedir que um atacante teste milhares de combinações de credenciais sem ser detectado ou bloqueado.

&#x20;      **A. Configuração do Servidor (vsftpd, ProFTPD)**

&#x20;     **.** Limitação de Tentativas e Tempo de Espera.

&#x20;     **. estrição de IP (Whitelisting)** Se possível, restrinja o acesso ao FTP apenas&#x20;

&#x20;    a IPs confiáveis (ex: rede interna ou VPN).

#### &#x20;Proteção contra Automação em Formulário Web

O objetivo é detectar e bloquear tentativas automatizadas de login em formulários HTML.

#### &#x20;Proteção contra Password Spraying em SMB/Active Directory

Password Spraying é difícil de detectar porque usa senhas comuns para muitos usuários, evitando bloqueios de conta. A defesa deve focar em **detecção de padrões** e **políticas de senha**.

**A. Políticas de Senha e Conta**

1. Política de Senha Robusta
2. Bloqueio de Conta Progressivo
3. MFA (Multi-Factor Authentication)
   * ```
     ```

<br>
