---
description: O Medusa exibirá a saída no terminal. Credenciais válidas são destacadas.
---

# Análise dos Resultados

**Exemplo de Saída:**

```
textDownloadCopy codeMedusa v3.12 [192.168.56.10]
...
[+] Host: 192.168.0.10  User: msfadmin  Pass: msfadmin
```

#### Interpretação

* **`msfadmin:msfadmin`**: Credencial padrão do Metasploitable 2.
* **`root:123456`**: Senha fraca configurada para o usuário root no sistema de teste.

#### Validação do Acesso

Para confirmar o sucesso do ataque, tente autenticar-se manualmente no SSH:

```
bashDownloadCopy codessh msfadmin@192.168.0.10
# Senha: msfadmin
```

Se o login for bem-sucedido, o brute force foi efetivo.
