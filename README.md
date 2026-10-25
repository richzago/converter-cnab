# Conversor CNAB

Aplicação web desenvolvida com **Python, Django e Django REST Framework**, com persistência em **PostgreSQL**, para importar transações de um arquivo texto em um **layout CNAB simplificado e específico**, registrar os dados e apresentar totais por estabelecimento.

> **Aviso importante:** este projeto é demonstrativo/educacional. **Não é um conversor universal de CNAB** e não deve ser usado para conciliação financeira de produção sem validação, testes e adaptações.

## 1. O que o projeto faz

- Recebe um arquivo `.txt` por formulário web.
- Lê registros de tamanho fixo, interpreta campos de transação e salva os dados no PostgreSQL.
- Apresenta uma página com o total agregado por estabelecimento, calculado na memória do processo Django na implementação atual.
- Possui interface visual **Violet Finance**, com templates Django.

**Rotas conhecidas:**

| Endereço | Função |
| --- | --- |
| `http://127.0.0.1:8000/api/cnab/` | Envio do arquivo |
| `http://127.0.0.1:8000/api/cnab/transactions/` | Resumo por estabelecimento |

> Os endereços acima dependem da configuração das rotas no arquivo `_core/urls.py`. O código da aplicação define `cnab/` e `cnab/transactions/`; o prefixo `/api/` é aplicado pelo projeto principal.

## 2. Tecnologias e versões

| Componente | Versão/observação |
| --- | --- |
| Python | **3.11.9 (recomendada e utilizada para executar o projeto)** |
| Django | **4.1.5** (versão observada na execução do projeto) |
| Django REST Framework | Utilizado nos serializers; confirmar versão no `requirements.txt` |
| PostgreSQL | Banco principal; confirmar versão local conforme instalação |
| HTML / CSS / JavaScript | Interface dos templates |
| Windows 11 + PowerShell | Ambiente usado nas instruções abaixo |

**Por que Python 3.11.9?** O projeto foi executado com sucesso nessa versão. A instalação inicial com uma versão mais recente do Python apresentou incompatibilidades de dependências. Portanto, **não utilize automaticamente o Python padrão do computador** sem conferir a versão. Não há garantia de compatibilidade com Python 3.13 ou versões posteriores.

O `requirements.txt` é a fonte de verdade das dependências do repositório. Não substitua suas versões sem necessidade.

## 3. Estrutura do projeto

```text
converter-cnab/
├── _core/
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── transactions/
│   ├── migrations/
│   ├── templates/
│   │   └── main/
│   │       ├── base.html
│   │       ├── main.html
│   │       ├── upload.html
│   │       └── transactions.html
│   ├── forms.py
│   ├── models.py
│   ├── serializers.py
│   ├── urls.py
│   └── views.py
├── .env.example
├── .gitignore
├── manage.py
├── README.md
└── requirements.txt
```

A pasta `.venv/` e o arquivo `.env` são **locais** e não devem ser versionados.

## 4. Pré-requisitos

1. Git, para clonar o repositório (ou baixar o ZIP do projeto).
2. **Python 3.11.9** instalado.
3. PostgreSQL instalado e em execução.
4. Um editor, como VS Code.
5. PowerShell, para executar os comandos do Windows.

### 4.1. Instalar Python 3.11.9

Baixe o Python 3.11.9 no site oficial: <https://www.python.org/downloads/release/python-3119/>.

Caso o instalador tradicional não esteja disponível para seu sistema, utilize o gerenciador oficial do Python para Windows e instale a versão 3.11.9 por ele. A sintaxe do gerenciador pode variar conforme a versão instalada.

Depois da instalação, abra **um novo PowerShell** e verifique:

```powershell
py -0p
```

O comando lista as versões e caminhos disponíveis quando o **Python Launcher (`py`)** está instalado. Procure uma instalação da série **3.11**.

Teste explicitamente:

```powershell
py -3.11 --version
```

O resultado esperado é `Python 3.11.9` se essa versão foi instalada. Se `py` não for reconhecido, veja a seção **Solução de problemas**.

> **Não basta executar `python --version` e presumir que é a versão correta.** Em computadores com várias versões, `python` pode apontar para o Python 3.13 ou outro interpretador.

### 4.2. Instalar PostgreSQL

Instale o PostgreSQL pelo site oficial: <https://www.postgresql.org/download/windows/>.

Durante a instalação, anote a senha atribuída ao usuário administrativo `postgres` e mantenha a porta padrão **5432**, se possível. Confirme que o serviço do PostgreSQL está em execução.

## 5. Instalação do projeto no Windows

### 5.1. Obter o código

Clone o repositório, substituindo o endereço pelo URL real do projeto:

```powershell
git clone <URL-DO-REPOSITORIO>
cd converter-cnab
```

Ou extraia o ZIP do repositório e abra a pasta que contém `manage.py` no VS Code.

### 5.2. Criar o ambiente virtual com a versão certa

**Este é o passo mais importante para evitar os problemas de versão.** Dentro da pasta do projeto:

```powershell
py -3.11 -m venv .venv
```

Ative o ambiente:

```powershell
.\.venv\Scripts\Activate.ps1
```

O terminal normalmente passa a mostrar `(.venv)` no início da linha.

Confirme que o executável é o do ambiente virtual e a versão é 3.11:

```powershell
python --version
python -c "import sys; print(sys.executable)"
```

O caminho deve terminar em algo como `converter-cnab\.venv\Scripts\python.exe`.

Se o PowerShell bloquear a ativação por política de execução, use **somente na sessão atual**:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

**Alternativa sem ativar o ambiente:** execute diretamente `\.venv\Scripts\python.exe` em todos os comandos (exemplo: `\.venv\Scripts\python.exe manage.py runserver`).

> Se você já criou `.venv` com Python 3.13, não é suficiente ativá-lo depois de instalar o 3.11. É necessário **recriar o ambiente** com a versão correta. Não exclua uma pasta de ambiente que contenha dados ou personalizações importantes sem verificar antes.

### 5.3. Instalar dependências

Com o ambiente virtual ativo:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Se ocorrer erro de compilação ou incompatibilidade, verifique primeiro `python --version`, o caminho do interpretador e as versões fixadas em `requirements.txt`. Não instale ou atualize pacotes aleatoriamente para contornar erros sem investigar.

## 6. Criar o banco de dados

No **pgAdmin** ou no `psql`, crie um banco com o nome usado nas instruções:

```sql
CREATE DATABASE convertercnab;
```

O usuário `postgres` precisa ter acesso ao banco. Caso utilize outro usuário, ajuste as variáveis do `.env` e as permissões no PostgreSQL.

Para verificar a conexão pelo `psql` (se estiver disponível no PATH):

```powershell
psql -h 127.0.0.1 -p 5432 -U postgres -d convertercnab
```

A senha será solicitada pelo PostgreSQL.

## 7. Configurar variáveis de ambiente

Na raiz do projeto, crie `.env` a partir de `.env.example`:

```powershell
Copy-Item .env.example .env
```

Se o `.env.example` estiver incompleto, confira se `.env` contém **as chaves efetivamente usadas no `_core/settings.py`**:

```dotenv
SECRET_KEY=coloque-uma-chave-secreta-local-aqui
POSTGRES_USERNAME=postgres
POSTGRES_PASSWORD=sua_senha_do_postgresql
POSTGRES_DB_NAME=convertercnab
```

A configuração de banco utilizada no projeto é equivalente a:

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.getenv("POSTGRES_DB_NAME"),
        "USER": os.getenv("POSTGRES_USERNAME"),
        "PASSWORD": os.getenv("POSTGRES_PASSWORD"),
        "HOST": "127.0.0.1",
        "PORT": "5432",
    },
}
```

O projeto também pode conter uma conexão SQLite secundária chamada `option_sqlite`, mas **a conexão padrão é PostgreSQL**.

**Segurança:** não publique `.env`, senha do banco nem `SECRET_KEY`. Confira se `.gitignore` contém `.env` e `.venv/`. A chave secreta deve ser exclusiva do seu ambiente.

> O projeto deve carregar as variáveis do `.env` no `settings.py` (por exemplo, com a biblioteca que já consta nas dependências). Se não carregar, as variáveis ficarão vazias e a conexão falhará. Confirme a implementação existente em vez de copiar configurações incompatíveis.

## 8. Preparar as tabelas

Com PostgreSQL iniciado, `.env` configurado e `.venv` ativo:

```powershell
python manage.py check
python manage.py migrate
```

O projeto já possui a migração inicial de `transactions`. Em uma instalação nova, normalmente **não é necessário** executar `makemigrations`, pois as migrações já fazem parte do código.

## 9. Iniciar a aplicação

```powershell
python manage.py runserver
```

Abra no navegador:

- **Upload:** <http://127.0.0.1:8000/api/cnab/>
- **Transações:** <http://127.0.0.1:8000/api/cnab/transactions/>

Para parar o servidor, pressione `Ctrl + C` no terminal.

O servidor `runserver` é destinado ao **desenvolvimento local**, não à produção.

## 10. Como importar um arquivo

1. Acesse a página de upload.
2. Escolha um arquivo `.txt` compatível com o layout abaixo.
3. Clique em **Processar arquivo**.
4. Confira a página de transações e os totais por estabelecimento.

### 10.1. Layout aceito pelo parser atual

O parser foi desenvolvido para **registros de largura fixa de 80 caracteres**, sem separadores entre os campos:

| Campo | Posições Python | Tamanho | Exemplo/formato |
| --- | --- | ---: | --- |
| Tipo | `[0:1]` | 1 | Dígito de 1 a 9 |
| Data | `[1:9]` | 8 | `AAAAMMDD` |
| Valor | `[9:19]` | 10 | Inteiro em centavos, com zeros à esquerda |
| CPF | `[19:30]` | 11 | 11 caracteres |
| Cartão | `[30:42]` | 12 | 12 caracteres |
| Hora | `[42:48]` | 6 | `HHMMSS` |
| Proprietário da loja | `[48:62]` | 14 | Texto com preenchimento |
| Nome da loja | `[62:80]` | 18 | Texto com preenchimento |

A soma é **80 caracteres por registro**. A implementação original utiliza `field[62:81]` para o nome da loja, mas como a fatia total possui 80 caracteres, somente os índices 62 a 79 estão disponíveis. Portanto, o campo efetivo é de **18 caracteres**, embora o modelo permita até 19.

Exemplo de montagem de **um registro fictício** em Python:

```python
registro = (
    "1"                  # tipo: 1
    "20261025"           # data: AAAAMMDD
    "0000002590"         # valor: R$ 25,90
    "00000000000"        # CPF fictício
    "000000000000"       # cartão fictício
    "143000"             # hora: 14:30:00
    "LOJA TESTE    "     # proprietário: 14 caracteres
    "MERCADO PAMPA     " # nome: 18 caracteres
)
assert len(registro) == 80
```

Os valores e documentos do exemplo são **fictícios**, não servem como dados reais.

### 10.2. Atenção ao formato do arquivo

- O conteúdo deve ser texto legível em **UTF-8**, conforme a decodificação atual.
- O parser remove `\r` e `\n` antes de separar os registros em blocos de 80 caracteres.
- Não inclua cabeçalhos, trailers, delimitadores ou caracteres adicionais que não façam parte do layout.
- O arquivo **não pode ser um documento de definição de layout bancário**, como um arquivo Protheus/TOTVS com linhas descritivas; isso causa erros ao interpretar datas e valores.
- A extensão `.txt` por si só **não garante** compatibilidade.

### 10.3. Problema conhecido: último registro

Na implementação original havia:

```python
while len(formated_file) > end:
```

Isso ignora o último registro quando o comprimento do arquivo é múltiplo exato de 80. A correção mínima é:

```python
while len(formated_file) >= end:
```

**Recomendação:** corrigir essa condição e usar arquivos compostos exclusivamente por registros completos, sem preenchimentos extras. A validação ideal também deve rejeitar arquivos cujo comprimento não seja múltiplo de 80.

> Se a versão atual de `views.py` ainda usar `>`, ela poderá ignorar o último registro. Confirme o código instalado antes de testar. Arquivos de teste antigos com um caractere extra de preenchimento foram um contorno temporário e **não devem ser tratados como formato oficial**.

## 11. Limitações conhecidas

Esta seção descreve comportamentos observados no código compartilhado durante a recuperação do projeto. Algumas limitações podem ter sido corrigidas localmente; verifique o código atual.

| Limitação | Impacto | Melhoria sugerida |
| --- | --- | --- |
| Layout fixo de 80 caracteres | Arquivos CNAB de outros bancos ou padrões não funcionam | Implementar parser por layout, versão e instituição |
| Totais armazenados em `stores`, dicionário global | Reiniciar o servidor zera os totais exibidos; diferentes processos podem divergir | Calcular agregações diretamente do banco |
| Reenvio do mesmo arquivo | Pode gerar transações duplicadas | Detectar duplicidade ou registrar lotes importados |
| Conversão do valor por `float` | Pode introduzir imprecisão monetária | Usar `Decimal` ou centavos inteiros |
| Falha no meio da importação | Registros anteriores podem ficar salvos mesmo com erro posterior | Validar todos os registros e usar transação atômica |
| Ausência de validação completa do arquivo | Arquivos incompatíveis podem gerar erro HTTP 500 | Validar extensão, tamanho, codificação, formato e conteúdo |
| Tratamento limitado de datas e horários | Dados inválidos causam exceções | Retornar mensagens de erro por linha |
| Sem controle de acesso observado | Não é adequado expor dados financeiros em produção | Implementar autenticação e autorização |
| Sem garantias de conciliação ou integridade bancária | Não é apropriado para operações financeiras reais | Criar testes e regras de negócio específicas |
| Dependências antigas | Podem não funcionar em versões modernas do Python | Fixar versões compatíveis e planejar atualizações |

### Observação sobre os totais

O código atual mantém os valores em memória por meio de `stores = {}`. **As transações são persistidas no PostgreSQL**, mas o resumo exibido na página **não é reconstruído automaticamente a partir do banco**. Isso significa que, após reiniciar o servidor, o resumo pode aparecer vazio, mesmo que existam registros salvos. Para uma solução definitiva, os totais precisam ser calculados com consultas e agregações no banco.

## 12. Solução de problemas

### `py` não é reconhecido

O Python Launcher pode não estar instalado ou acessível. Verifique:

```powershell
where.exe python
python --version
```

Se você usa o **Python Install Manager** moderno, consulte os comandos de listagem e execução da sua instalação (por exemplo, `py list` quando disponível) e selecione explicitamente o Python 3.11.9. Outra opção é chamar o `python.exe` da instalação 3.11 pelo **caminho completo** para criar o ambiente virtual:

```powershell
& "CAMINHO_COMPLETO_PARA_PYTHON_3_11\python.exe" -m venv .venv
```

Substitua o caminho pelo executável real; **não use um caminho de exemplo literalmente**.

### `python --version` mostra Python 3.13

Você provavelmente está fora do ambiente virtual ou criou `.venv` com a versão errada. Verifique `python -c "import sys; print(sys.executable)"`. Recrie o ambiente com o interpretador 3.11.9 e reinstale as dependências, se necessário.

### PowerShell bloqueia `Activate.ps1`

Use o ajuste temporário de sessão descrito na seção 5.2 ou execute diretamente `.venv\Scripts\python.exe`.

### Erro de conexão com PostgreSQL

Confirme que o serviço está em execução, que o banco `convertercnab` existe, que o usuário e a senha do `.env` estão corretos e que a porta é `5432`. Uma senha esquecida precisa ser redefinida no PostgreSQL; **não tente recuperá-la do código**.

### `TemplateDoesNotExist: upload.html`

Os templates estão em `transactions/templates/main/`. Portanto, em `views.py`, utilize:

```python
return render(request, "main/upload.html", {"form": form})
```

E para a página de resultados:

```python
return render(request, "main/transactions.html", context)
```

Nos templates `upload.html`, `transactions.html` e `main.html`, a herança correta é:

```django
{% extends "main/base.html" %}
```

Confira também `APP_DIRS=True` em `TEMPLATES` e `transactions` em `INSTALLED_APPS`.

### `TypeError: context must be a dict rather than module`

O terceiro argumento de `render` deve ser um **dicionário**, como `{"form": form}`, e não um módulo importado. Revise o retorno da view `upload_file`.

### `ValueError: invalid literal for int() with base 10`

O conteúdo não corresponde ao layout esperado ou os blocos estão desalinhados. Verifique as posições, o tamanho dos registros e se o arquivo contém cabeçalhos ou trailers. Exemplos já observados foram `0D L` e `IM_A`, causados por arquivos incompatíveis ou texto extra interpretado como transação.

### A página de transações está vazia após reiniciar

É uma limitação do resumo em memória. Reiniciar o servidor não apaga automaticamente as transações salvas no PostgreSQL. Consulte a seção 11.

### O formulário não recebe o arquivo

O campo Django é chamado `file`. Se usar um `<input>` manual, ele precisa ter `name="file"` e o formulário precisa de `enctype="multipart/form-data"` e `{% csrf_token %}`. Na view, valide o formulário antes de acessar `form.cleaned_data["file"]`.

## 13. Recomendações para desenvolvimento futuro

1. Criar testes automatizados para o parser, inclusive casos inválidos e arquivo com apenas um registro.
2. Separar o processamento CNAB das views Django.
3. Calcular totais no PostgreSQL e registrar cada importação como um lote.
4. Adicionar detecção de duplicidade e importação atômica.
5. Tratar valores com `Decimal` e validar rigorosamente os campos.
6. Exibir erros amigáveis de importação, sem revelar traceback ao usuário.
7. Atualizar dependências com testes e planejar suporte a outros layouts CNAB.
8. Adicionar autenticação e controles de segurança antes de qualquer publicação real.

## 14. Escopo e uso responsável

O Conversor CNAB é um projeto de demonstração de habilidades em **Python, Django, processamento de arquivos, persistência de dados e construção de interfaces web**. Os arquivos de teste devem conter apenas dados fictícios. Não utilize informações bancárias ou pessoais reais sem controles apropriados.

---

**Nota de manutenção:** este README foi elaborado com base na estrutura e no código compartilhados durante a configuração do projeto. Antes de publicar, confira se as versões no `requirements.txt`, os comandos, o comportamento do `views.py` e os arquivos presentes no repositório correspondem à versão final do código.
