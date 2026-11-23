# Laboratório Prático: Gerenciamento de Usuários, Grupos e Permissões no Linux

## 🎯 Objetivo

Configurar o controle de acesso a um diretório compartilhado (`/srv/financeiro`) em ambiente Linux (Ubuntu/Debian), aplicando gerenciamento de usuários, grupos secundários, propriedade de arquivos (`chown`) e permissões octais com bit **SGID** (`chmod 2770`). O laboratório garante que apenas o usuário `administrador` e os integrantes do grupo `financeiro` (`cicrano` e `beltrano`) tenham acesso de leitura e escrita, bloqueando usuários de outros grupos (`fulano`, do grupo `devs`) e usuários comuns (`novato`).

---

## 🛠️ Procedimento e Evidências

### 1. Criação dos Usuários de Teste

Para simular os diferentes perfis de acesso exigidos no laboratório, foram criadas as contas de usuários utilizando o utilitário interativo `sudo adduser <usuario>`. Esse comando cria o registro do usuário em `/etc/passwd`, gera automaticamente um grupo primário homônimo, cria o diretório pessoal em `/home/<usuario>` copiando os arquivos base de `/etc/skel` e solicita a definição de senha e informações complementares (GECOS).

* **Como foi obtido:** Execução sequencial de `sudo adduser` para `fulano` (UID/GID `1001`), `cicrano` (UID/GID `1002`), `beltrano` e `novato` (UID/GID `1004`), confirmando os dados padrão ao final de cada criação.

![Criação dos usuários fulano e cicrano via comando adduser no terminal](imagens/criar.png)

![Finalização do cadastro de beltrano e criação do usuário novato via adduser](imagens/criar2.png)

---

### 2. Criação do Grupo `financeiro`

Em seguida, foi criado o grupo departamental que centralizará as permissões de escrita e leitura da pasta compartilhada.

* **Como foi obtido:** Executou-se `sudo groupadd financeiro` para registrar o novo grupo no sistema (`/etc/group`). A validação foi feita com `getent group financeiro`, que consultou a base de grupos do sistema e retornou `financeiro:x:1005:`, confirmando a criação com o **GID 1005**. *(Nota: na sequência imediata, o comando `sudo usermod -aG cicrano` foi executado sem informar o nome do grupo, fazendo o terminal exibir a documentação de sintaxe do comando).*

![Criação do grupo financeiro com groupadd e validação do GID 1005 com getent](imagens/grupo.png)

---

### 3. Associação de Usuários ao Grupo e Criação do Diretório `/srv/financeiro`

Com o grupo criado, os usuários `cicrano` e `beltrano` foram adicionados ao grupo `financeiro` como grupo secundário, preservando o vínculo prévio de ambos com o grupo `devs`. Na mesma etapa, criou-se o diretório compartilhado em `/srv/financeiro`.

* **Como foi obtido:**
  * `sudo usermod -aG financeiro cicrano` e `sudo usermod -aG financeiro beltrano`: as opções `-a` (*append*) e `-G` (*groups*) anexam o grupo `financeiro` aos usuários sem sobrescrever seus grupos secundários existentes.
  * `sudo mkdir -p /srv/financeiro`: cria o diretório alvo dentro de `/srv`.
  * `ls -ld /srv/financeiro`: exibe os metadados do diretório recém-criado, mostrando inicialmente o dono padrão `root:root` e permissões `drwxr-xr-x` (`755`).

![Associação de cicrano e beltrano ao grupo financeiro e criação da pasta /srv/financeiro](imagens/pastacomp.png)

---

### 4. Definição de Proprietário e Permissões de Acesso (`2770`)

Para restringir o acesso exclusivamente ao `administrador` e aos membros do grupo `financeiro`, alterou-se a posse do diretório e aplicou-se a máscara de permissão `2770`.

* **Como foi obtido:**
  * `sudo chown administrador:financeiro /srv/financeiro`: define `administrador` como usuário proprietário e `financeiro` como grupo proprietário do diretório.
  * `sudo chmod 2770 /srv/financeiro`: aplica o bit especial **SGID (`2`)**, permissão total (`7` = `rwx`) para o dono e para o grupo, e bloqueio total (`0` = `---`) para outros usuários.
  * `ls -ld /srv/financeiro`: confirma a aplicação na saída `drwxrws--- 2 administrador financeiro`, onde o caractere `s` no lugar do `x` do grupo indica que o **SGID** está ativo (novos arquivos herdarão automaticamente o grupo `financeiro`) e `---` garante que terceiros não têm qualquer acesso.

![Alteração de posse para administrador:financeiro e aplicação da permissão 2770](imagens/permisoes.png)

---

### 5. Testes Práticos de Validação no Terminal

Para comprovar o funcionamento das regras de segurança, foram realizadas sessões de teste autenticadas com cada perfil de usuário.

#### 5.1. Teste com `cicrano` (Membro do grupo `financeiro` — Acesso Permitido)

* **Como foi obtido:** Autenticado como `cicrano`, executou-se `echo "Relatorio financeiro - Versão 1" > /srv/financeiro/relatorioo.txt` para criar o arquivo e, em seguida, `echo "alteração financeiro - cicrano" > /srv/financeiro/relatorioo.txt` para modificar seu conteúdo. A leitura com `cat /srv/financeiro/relatorioo.txt` exibiu `alteração financeiro - cicrano`, comprovando que membros do grupo `financeiro` possuem permissão de criação, edição e leitura na pasta.

![Teste de criação, alteração e leitura de relatório realizado com sucesso por cicrano](imagens/teste1.png)

#### 5.2. Teste com `fulano` (Membro do grupo `devs` — Acesso Negado)

* **Como foi obtido:** Utilizou-se `su - fulano` para abrir uma sessão de login limpa com as credenciais de `fulano`. Ao tentar listar o diretório (`ls /srv/financeiro`) e criar um arquivo (`touch /srv/financeiro/teste_fulano.txt`), o sistema retornou `Permission denied` em ambos os comandos, pois `fulano` pertence ao grupo `devs` e recai na categoria *Outros* (`---`).

![Bloqueio com Permission denied ao tentar listar e criar arquivos usando o usuário fulano](imagens/teste2.png)

#### 5.3. Teste com `novato` (Usuário sem vínculo ao grupo — Acesso Negado)

* **Como foi obtido:** Utilizou-se `su - novato` para assumir a sessão do usuário `novato`. Tanto a tentativa de listagem (`ls /srv/financeiro`) quanto a tentativa de leitura direta (`cat /srv/financeiro/relatorio.txt`) resultaram em `Permission denied`, validando o isolamento completo do diretório.

![Bloqueio com Permission denied ao tentar listar e ler arquivos usando o usuário novato](imagens/teste3.png)

---

## 📝 Questões e Respostas

### 1. Como o grupo `financeiro` foi criado e como validar sua criação?
O grupo foi criado por meio do comando administrativo `sudo groupadd financeiro`, que adiciona uma nova entrada no arquivo `/etc/group`. A validação foi obtida executando `getent group financeiro`, cuja saída `financeiro:x:1005:` confirma que o grupo está ativo no sistema sob o identificador numérico (**GID**) `1005`.

### 2. Por que utilizar `usermod -aG` ao associar `cicrano` e `beltrano` ao novo grupo?
No Linux, o comando `usermod -G <grupo> <usuario>` (sem a flag `-a`) substitui toda a lista de grupos secundários do usuário, o que removeria `cicrano` e `beltrano` do grupo `devs`. O uso combinado de `-aG` (*append* + *groups*) garante que o grupo `financeiro` seja acrescentado à conta mantendo intactas as associações anteriores.

| Comando / Parâmetro | Função Técnica |
| :--- | :--- |
| `groupadd financeiro` | Cria o novo grupo `financeiro` no banco de grupos do sistema. |
| `usermod -aG` | Anexa (`-a`) um grupo secundário (`-G`) ao usuário sem remover seus grupos atuais. |
| `mkdir -p /srv/financeiro` | Cria o diretório compartilhado e eventuais diretórios pais necessários. |
| `chown administrador:financeiro` | Define simultaneamente o usuário dono (`administrador`) e o grupo dono (`financeiro`). |
| `chmod 2770` | Define bit SGID (`2`), `rwx` para dono (`7`), `rwx` para grupo (`7`) e sem acesso para outros (`0`). |

### 3. Como funciona a permissão octal `2770` aplicada em `/srv/financeiro`?
A permissão `2770` divide-se em quatro dígitos octais que controlam o comportamento colaborativo e o isolamento do diretório:

| Dígito | Escopo | Representação Simbólica | Efeito Prático no Diretório `/srv/financeiro` |
| :---: | :--- | :---: | :--- |
| **`2`** | **Bit Especial (SGID)** | `s` (no grupo) | Novos arquivos/subpastas criados herdam automaticamente o grupo `financeiro`. |
| **`7`** | **Dono (`administrador`)** | `rwx` | Controle total: leitura, gravação, criação, renomeação e exclusão. |
| **`7`** | **Grupo (`financeiro`)** | `rwx` | Acesso colaborativo: `cicrano` e `beltrano` podem listar, criar e editar arquivos. |
| **`0`** | **Outros (`devs` / `novato`)** | `---` | Bloqueio absoluto: impede listar conteúdo, ler arquivos ou gravar dados. |

### 4. Qual foi o resultado comparativo dos testes de acesso entre os usuários?
Os testes realizados no terminal comprovaram que a política de permissões atende integralmente aos requisitos de segurança solicitados:

| Usuário | Grupo(s) Relevante(s) | Classificação no Diretório | Comandos Executados no Teste | Resultado Obtido |
| :--- | :--- | :--- | :--- | :--- |
| **`administrador`** | `administrador` | Dono (`rwx`) | Configurado via `chown` | Controle total sobre o diretório. |
| **`cicrano`** | `devs`, `financeiro` | Grupo (`rws`) | `echo "..." > relatorioo.txt` e `cat` | **Sucesso** (criou, alterou e leu o arquivo). |
| **`beltrano`** | `devs`, `financeiro` | Grupo (`rws`) | Associado via `usermod -aG` | Mesmas permissões de leitura/escrita de `cicrano`. |
| **`fulano`** | `devs` | Outros (`---`) | `ls` e `touch teste_fulano.txt` | **`Permission denied`** em ambas as operações. |
| **`novato`** | `novato` | Outros (`---`) | `ls` e `cat relatorio.txt` | **`Permission denied`** em ambas as operações. |
