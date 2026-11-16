---
description: >-
  O objetivo é impedir que um atacante teste milhares de combinações de
  credenciais sem ser detectado ou bloqueado.
---

# Considerações Finais

Para se proteger efetivamente dessas três vetores de ataque, é necessário implementar camadas de defesa que combinam autenticação robusta, monitoramento de rede e lógica de aplicação. Abaixo, detalho as estratégias para cada cenário específico.

#### 1. Proteção contra Força Bruta em Servidor FTP

O FTP tradicional é inseguro por design. A defesa deve focar em criptografia, limitação de tentativas e isolamento.

* **Use SFTP/FTPS em vez de FTP:**
  * **SFTP (SSH File Transfer Protocol):** Roda sobre SSH, o que permite aproveitar os recursos de autenticação do SSH (chaves públicas, multi-fator).
  * **FTPS:** Adiciona TLS ao FTP tradicional. É menos seguro que SFTP, mas compatível com clientes antigos.
* **Autenticação com Chaves Públicas:**
  * Desabilite senhas e use apenas chaves SSH. Isso elimina a superfície de ataque para _brute-force_ de senhas.
* **Fail2Ban / Controladores de Taxa:**
  * Implemente `fail2ban` para monitorar logs do servidor (ex: `vsftpd` ou `proftpd`) e banir IPs automaticamente após N tentativas falhas.
  *   Exemplo de configuração básica no `jail.local`:

      ```
      iniDownloadCopy code[vsftpd]
      enabled  = true
      filter   = vsftpd
      logpath  = /var/log/auth.log
      maxretry = 5
      bantime  = 1h
      ```
* **Restrição de Rede (IP Allowlist):**
  * Use `iptables` ou `nftables` para permitir conexões FTP/SFTP apenas de IPs ou sub-redes confiáveis.
  *   Exemplo `nftables`:

      ```
      nftDownloadCopy codeinet filter {
          input {
              tcp dport 22 ct state new ip saddr { 10.0.0.0/8, 192.168.1.0/24 } counter accept
              tcp dport 22 counter drop
          }
      }
      ```
* **Isolamento de Usuários (Chroot):**
  * Configure o servidor FTP para chroot cada usuário em seu diretório home, impedindo navegação no sistema.

#### 2. Proteção contra Automação de Tentativas em Formulário Web

Ataques de _credential stuffing_ ou _brute-force_ em login de web apps exigem defesas no nível da aplicação e da rede.

* **Rate Limiting (Limitação de Taxa):**
  *   **Camada de Rede (Nginx/Apache):** Limite requisições por IP.

      ```
      nginxDownloadCopy codelimit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

      location /login {
          limit_req zone=login burst=10 nodelay;
      }
      ```
  * **Camada de Aplicação:** Implemente lógica no backend (ex: Redis) para rastrear tentativas falhas por usuário ou IP.
* **CAPTCHA Inteligente:**
  * Use serviços como hCaptcha, reCAPTCHA v3 ou Turnstile. O reCAPTCHA v3 é ideal pois atribui um "score" de confiança sem interromper o usuário legítimo.
* **Exponencial Backoff e Lockout Temporal:**
  * Após 3 falhas, exija CAPTCHA.
  * Após 5 falhas, bloqueie o usuário por 15 minutos.
  * Após 10 falhas, bloqueie por 24 horas ou exija verificação de e-mail.
* **Monitoramento e Alertas:**
  * Registre todos os tentativas de login (sucesso/falha) com metadados (IP, User-Agent, horário).
  * Envie alertas para SIEM (ex: Splunk, Elastic) se houver padrões anômalos (ex: 50 falhas de 10 IPs diferentes em 1 minuto).
* **Session Management:**
  * Use tokens de sessão seguros (HttpOnly, Secure, SameSite=Strict).
  * Implemente MFA (Multi-Fator Autenticação) para usuários críticos.

#### 3. Proteção contra Password Spraying em SMB com Enumeração

O _password spraying_ ataca múltiplos usuários com uma senha comum, enquanto a enumeração revela usernames válidos. A defesa deve focar em ocultar informações e detectar padrões.

* **Desabilitar Enumeração de Usuários:**
  * **Windows (GPO):**
    * `Dont enumerate users and groups`: Impede que `net user` liste contas.
    * `Restrict anonymous SAM accounts name enumeration`: Impede a enumeração via SMB.
    * `Enable network access: Let Everyone permissions apply to user accounts only`: Evita enumeração de grupos.
  * **Linux (Samba):**
    *   Em `/etc/smb.conf`:

        ```
        iniDownloadCopy code[global]
            map to guest = Bad User
            restrict anonymous = 2
            hide unreadable = yes
            hide writeable = yes
            username map = /etc/samba/username_map.txt
        ```
* **Account Lockout Policy (Política de Bloqueio de Conta):**
  * Configure um limiar de bloqueio (ex: 10 tentativas falhas em 30 minutos) para bloquear a conta por 15 minutos.
  * **Atenção:** Isso pode ser usado por atacantes para DDoS suas contas legítimas (_account lockout DoS_). Monitore padrões de bloqueio.
* **Monitoramento de Logon (Event Log Analysis):**
  * **Windows:** Monitore eventos `4625` (falha de login) e `4672` (privilégios especiais).
  * **Padrão de Spraying:** Muitas falhas de login (4625) com o mesmo erro (`0xC000006D` = senha incorreta) para **diferentes** usuários a partir do mesmo IP ou sub-rede.
  *   **Regra de Detecção (SIEM):**

      ```
      kqlDownloadCopy code// Exemplo de lógica KQL para Azure Sentinel/Splunk
      SecurityEvent
      | where EventID == 4625
      | where LogonType == 3
      | summarize count() by IpAddress, Account
      | where count() > 5
      ```
* **Implementar MFA para SMB (NLA):**
  * Use **Network Level Authentication (NLA)** para exigir autenticação antes de estabelecer a sessão SMB.
  * Em ambientes corporativos, use **Kerberos** com **Mutual Authentication** para evitar ataques de _pass-the-hash_ e _relay_.
* **Segmentação de Rede:**
  * Use firewalls para restringir acesso SMB (porta 445) apenas a hosts administrativos ou sistemas críticos.
  * Bloqueie SMBv1 e force SMBv3 com cifragem (AES-128-CCM).

#### Resumo de Estratégias Cruzadas

| Técnica           | FTP        | Web Form         | SMB            |
| ----------------- | ---------- | ---------------- | -------------- |
| **Rate Limiting** | Fail2Ban   | Nginx limit\_req | IPS/IDS ou NLA |
| **Autenticação**  | Chaves SSH | MFA + Sessões    | Kerberos + NLA |
| **Ocultação**     | Chroot     | Rate Limiting    | GPO/Samba hide |
| **Detecção**      | Logs Auth  | Logs Aplicação   | Event ID 4625  |
| **Criptografia**  | TLS (FTPS) | HTTPS            | SMBv3 AES      |

#### Boas Práticas Gerais

1. **Logs Centralizados:** Envie todos os logs de autenticação para um SIEM.
2. **Testes de Invasão:** Realize _red team_ exercícios para validar as defesas.
3. **Atualização de Software:** Mantenha servidores FTP, web servers e AD/Samba atualizados contra vulnerabilidades conhecidas.
4. **Princípio do Menor Privilégio:** Garanta que cada usuário/serviço tenha apenas as permissões necessárias.
