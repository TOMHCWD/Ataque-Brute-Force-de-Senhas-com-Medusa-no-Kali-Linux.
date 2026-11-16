# Criação dos Wordlists

Para otimizar o ataque, criaremos dois arquivos de texto contendo os possíveis usuários e senhas.

#### 1. Passo a passo para criar Lista de Usuários:

No terminal do Kali Linux execute o seguinte comando.

* **echo -e "user\nmsfadmin\nadmin\nroot > users.tx**t
* caso desejar acresentar mais usuarios a lista execute o comando

&#x20;     no terminal, neste caso vou usar o NANO.

* **nano users.txt** (abaixo segue alguns nomes de usuário).

#### `user.txt` (Lista de Usuários)

Este arquivo contém usernames comuns encontrados em sistemas Unix/Linux.

**Conteúdo do arquivo `user.txt`:**

```
root
admin
administrator
user
guest
test
msfadmin
oracle
postgres
mysql
www
http
ftp
mail
backup
support
```

#### &#x20; 3. `senhas.txt` (Lista de Senhas).

No terminal do Kali Linux execute o seguinte comando.

* **echo -e "123456\npassword\nrequest\nmsfadmin > senhas.tx**t
* caso desejar acresentar mais senhas a lista execute o comando

&#x20;     no terminal, neste caso vou usar o NANO.

* **nano senha.txt** (abaixo segue alguns nomes de usuário).

```
123456
admin
administrator
password
senha
teste
msfadmin
oracle
postgres
mysql
msfadmin
```
