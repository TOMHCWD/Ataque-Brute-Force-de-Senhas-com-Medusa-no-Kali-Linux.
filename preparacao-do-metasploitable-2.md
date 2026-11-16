# Preparação do Metasploitable 2

Preparação do Metasploitable 2

1. **Download:** Baixe a imagem do Metasploitable 2 no site [https://sourceforge.net/projects/metasploitable/](https://sourceforge.net/projects/metasploitable/)
2. **Importação:** Importe a máquina no VirtualBox.
3. **Configuração de Rede:**
   * Ajuste a interface de rede para **Host-Only Adapter**.
   * Defina o IP estático `192.168.0.10`.
4. **Inicialização:**
   * Inicie a VM.
   * Login padrão: `msfadmin` / `msfadmin`.
   * O serviço FTP deve estar ativo.

> **Verificação de Conectividade:** A partir do Kali Linux, verifique se o alvo está acessível:
>
> ```bash
> ping 192.168.0.7
>
> ```

5. **Video da Instalação:**

&#x20;       [**https://www.youtube.com/watch?v=MEd8diEWbw0\&t=51s**](https://www.youtube.com/watch?v=MEd8diEWbw0\&t=51s)
