# Baixe — guia das ferramentas MIATSUNO-Lab

Guia em português para instalar e usar as ferramentas de terminal publicadas no [MIATSUNO-Lab](https://github.com/MIATSUNO/MIATSUNO-Lab). O repositório `baixe` reúne documentação; o código das ferramentas continua no MIATSUNO-Lab.

> O guia cobre os 17 comandos Python atualmente listados no MIATSUNO-Lab. Kayepad e Kachey são projetos separados, não ferramentas de terminal instaladas por este procedimento.

## Requisitos

- Android 7.0 ou posterior e Termux instalado por uma origem oficial. Use a mesma origem para atualizações e complementos; APKs de origens diferentes podem ter assinaturas incompatíveis.
- Conexão com a internet para instalar pacotes e clonar o código.
- As ferramentas rodam no Termux, mas são programas de terminal, não aplicativos com interface gráfica.

Para escolher um mirror oficial, rode `termux-change-repo`, selecione o repositório **Main** e escolha um mirror disponível. Não existe um mirror que seja sempre o mais rápido para toda operadora e região. Os canais `science` e `game` foram incorporados ao Main; `root-repo` e `x11-repo` são opcionais e não são necessários para estas ferramentas. Consulte a [gestão de pacotes](https://github.com/termux/termux-packages/wiki/package-management) e a [lista oficial de mirrors](https://github.com/termux/termux-packages/wiki/Mirrors).

## Instalação pelo Git

No Termux, execute:

```sh
pkg update && pkg upgrade -y
pkg install -y python git nano
cd "$HOME"
git clone https://github.com/MIATSUNO/MIATSUNO-Lab
cd "$HOME/MIATSUNO-Lab"
python -m venv .venv
source .venv/bin/activate
python -m pip install -r security/requirements.txt
bash automation/lookay/install.sh
```

O repositório é público: não precisa de token nem senha do GitHub. Python, Git e Nano são instalados pelo repositório oficial do Termux. As ferramentas usam a biblioteca padrão do Python, com exceção das ferramentas de segurança, que usam `requests`, instalada dentro de `.venv` pelo arquivo `security/requirements.txt`. Mantenha o ambiente virtual ativado enquanto executar esses scripts, ou invoque diretamente `./.venv/bin/python`.

O instalador `automation/lookay/install.sh` coloca o comando `lookay` em `$PREFIX/bin`. As outras ferramentas podem ser chamadas com `python caminho/do/script.py`. Para ver as opções completas de qualquer script, use `python caminho/do/script.py --help`.

### Acesso aos arquivos do Android

Só rode `termux-setup-storage` se quiser acessar Downloads ou outros arquivos compartilhados do Android. Aceite o pedido de acesso do próprio Android; isso cria atalhos, como `~/storage/downloads`. As ferramentas não precisam de localização, câmera, microfone, contatos, SMS, root ou acesso à sobreposição de tela. Para trabalhar com arquivos, é mais seguro manter cópias dentro do diretório privado do Termux ou usar apenas a pasta compartilhada necessária.

## Ferramentas

### Automação

#### `file_sift` — organizar arquivos por categoria

Classifica arquivos por extensão em pastas como documentos, imagens, código e outros. Cria um manifesto para desfazer os movimentos. Sem `--apply`, `organize` apenas simula.

```sh
python automation/file_sift.py organize ~/storage/downloads -r -d ~/storage/downloads_organizados --dry-run
```

Confira as linhas `origem -> destino`. Se o plano estiver correto, repita sem `--dry-run` e com `--apply`:

```sh
python automation/file_sift.py organize ~/storage/downloads -r -d ~/storage/downloads_organizados --apply
```

Guarde o manifesto. Antes de desfazer, confira o plano com `undo MANIFEST --dry-run`; `undo` e `restore` usam o manifesto para reverter movimentos. Não use `--overwrite` sem ter certeza: ele permite substituir um arquivo que já existe no caminho original. O destino precisa ficar fora da pasta de origem.

#### `site_watch` — acompanhar mudanças em páginas

Faz requisições HTTP(S) GET, calcula uma impressão digital da resposta e guarda estado local.

```sh
python automation/site_watch.py -s ~/site-watch.json https://example.org
```

Na primeira consulta, `NEW` significa que ainda não havia estado salvo; `UNCHANGED` significa que a resposta comparada não mudou; `CHANGED` indica mudança de conteúdo, status ou resposta. Isso não prova que houve um problema. `ERROR` indica falha de requisição. O estado fica no JSON indicado por `-s`. Use `--url-file urls.txt` para uma lista (uma URL por linha), `--json` para saída estruturada e `--fail-on-change` para retornar código de saída 3 quando algo for novo ou mudar. Só monitore sites que você tem autorização para consultar; não coloque cabeçalhos secretos na linha de comando, pois podem ficar no histórico do shell.

#### `backup_buddy` — backups ZIP completos e incrementais

Compara uma pasta com o estado salvo e cria arquivos ZIP verificados. A primeira execução cria a base completa; as seguintes podem ser incrementais.

```sh
python automation/backup_buddy.py ~/Documents ~/Backups --dry-run
```

Leia o plano `ADD`/`DELETE`. Retire `--dry-run` para gravar o backup. O arquivo de estado fica, por padrão, dentro do destino. `--full` cria um novo backup completo. Para simular uma restauração, use `python automation/backup_buddy.py rollback caminho/backup.zip --dry-run`.

A restauração real modifica a pasta de origem e pode remover arquivos atuais antes de reconstruir o estado do backup. Faça uma simulação primeiro e guarde uma cópia separada dos arquivos importantes. O destino do backup deve ficar fora da origem.

#### `lookay` — cliente de terminal da Kayepad

Depois de executar `bash automation/lookay/install.sh`, use:

```sh
lookay --help
lookay login
lookay status
lookay obras --search "autor ou palavra"
lookay obras --json
lookay escrever
lookay acompanhar https://example.org
lookay logout
```

`login` pede o e-mail e a senha no próprio Termux; a senha não aparece enquanto você digita. A sessão fica em `~/.config/lookay/session.json`: trate esse arquivo como segredo e não o envie a ninguém. `obras` consulta dados públicos da Veristante. `escrever` monta uma obra e salva um JSON local para importar; não publica nem envia a obra à Kayepad. `acompanhar` faz GETs somente de leitura e guarda estado local; informe apenas páginas que você tem permissão para consultar. `logout` remove a sessão local.

### Segurança defensiva

Use as três ferramentas seguintes apenas em serviços próprios ou com autorização explícita.

#### `header_guard` — cabeçalhos HTTP de segurança

```sh
python security/header_guard.py https://example.org
```

Faz um GET e verifica cabeçalhos como HSTS, CSP, `X-Content-Type-Options`, `Referrer-Policy` e `Permissions-Policy`. `Missing` mostra cabeçalhos ausentes; `Weak` indica uma configuração considerada fraca. Um código de saída 0 significa que a requisição funcionou e não foram encontrados itens ausentes/fracos segundo essas regras — não é uma certificação de segurança. Confira `Final URL` quando houver redirecionamento. `--json` inclui cabeçalhos de resposta; revise-os antes de compartilhar.

#### `dependency_lens` — metadados e vulnerabilidades de dependências

```sh
python security/dependency_lens.py security/requirements.txt --osv --timeout 15
```

Lê requisitos simples e consulta metadados públicos do PyPI. `--osv` pede também informações ao serviço OSV; a ferramenta não instala nem altera pacotes. O JSON mostra as dependências, as versões publicadas e, quando consultado, vulnerabilidades conhecidas. Uma lista vazia significa que aquela consulta não encontrou registros; não prova que o pacote seja seguro. Falhas de rede ou API aparecem como falhas, não como confirmação de segurança. Evite incluir índices privados ou segredos no arquivo consultado.

#### `pyroom` — checklist HTTP somente de leitura

```sh
python security/pyroom.py --list
python security/pyroom.py https://example.org --checks check_http_status,check_security_headers --timeout 10
```

`--list` mostra os nomes das 45 verificações disponíveis. A saída contém `passed` e `total`, além do resultado de cada verificação; leia os detalhes e erros individualmente. A contagem não é uma nota nem um certificado. A execução completa pode fazer várias requisições GET e OPTIONS a caminhos comuns. Não envia formulários, não explora falhas, não faz brute force e não grava dados no servidor, mas gera tráfego real: use apenas em alvos autorizados.

### Utilitários de programação

#### `env_doctor` — diagnóstico do ambiente local

```sh
python practical/env_doctor.py --executable git --path "$HOME/MIATSUNO-Lab" --environment HOME --json
```

Verifica executáveis, caminhos legíveis e presença de variáveis de ambiente. `--executable`, `--path` e `--environment` podem ser repetidos. A checagem de variável informa apenas se ela está definida, não revela o valor. `--url URL` faz um GET opcional; sem essa opção, não acessa a rede. Em texto, `ok` é o resumo; em JSON, examine cada lista e campo. O código de saída é 0 quando todas as verificações solicitadas passam e 1 se alguma falhar.

#### `git_scribe` — relatório somente de leitura de um repositório Git

```sh
python practical/git_scribe.py "$HOME/MIATSUNO-Lab" --limit 5 --json
```

Mostra raiz, branch, estado da árvore de trabalho, commits recentes e remotos. `clean: true` indica árvore limpa; alterações locais aparecem na seção `status`. A ferramenta não faz commit, checkout, fetch ou push.

#### `snippet_box` — guardar e buscar snippets locais

```sh
python practical/snippet_box.py --file ~/snippets.json add exemplo --text 'python3 --version' --tags termux,python
python practical/snippet_box.py --file ~/snippets.json get exemplo
python practical/snippet_box.py --file ~/snippets.json list --tag python
python practical/snippet_box.py --file ~/snippets.json search python
python practical/snippet_box.py --file ~/snippets.json export ~/snippets-backup.json
```

Também aceita `remove NAME` e `import ARQUIVO`. `get` e `remove` usam o nome exato. `import` combina registros por padrão; `import ARQUIVO --replace` substitui o banco inteiro pelo conteúdo importado. Exporte uma cópia antes de substituir ou remover algo importante. O banco é um JSON local; não guarde senhas, tokens ou chaves nele.

### Ferramentas geek

#### `solar_ledger` — nascer e pôr do Sol

```sh
python geek/solar_ledger.py --latitude -23.5505 --longitude -46.6333
python geek/solar_ledger.py --latitude -23.5505 --longitude -46.6333 --date 2026-10-19 --save ~/solar.json --json
```

Informe latitude e longitude em graus decimais; o exemplo usa aproximadamente São Paulo. A saída mostra nascer, pôr do Sol, duração do dia e fuso retornado pelo serviço. `--save` acrescenta o registro a um JSON local. As coordenadas e a data são enviadas ao Open-Meteo; não é necessário dar acesso ao GPS. Os horários são dados do serviço, não orientação para decisões legais ou emergenciais.

#### `curiosity_terminal` — pesquisar a Wikipédia no terminal

```sh
python geek/curiosity_terminal.py --language pt search 'história do rádio' --limit 3
python geek/curiosity_terminal.py --language pt random
```

`search` exibe títulos, trechos e links; `random` traz um artigo aleatório. `--language` usa códigos curtos como `pt` ou `en`, não `pt-BR`; `--limit` aceita de 1 a 50. `--json` gera dados estruturados. A consulta é enviada à Wikimedia; não pesquise texto privado. Verifique as fontes do artigo, pois resultados enciclopédicos não são automaticamente aconselhamento confiável.

### Ferramentas legadas

#### `column_fall` — jogo de encaixe no terminal

```sh
python legacy/column_fall/column_fall.py --demo
python legacy/column_fall/column_fall.py
```

`--demo` exibe os controles sem iniciar a partida. No jogo, `A` e `D` movem, `S` desce devagar, espaço derruba, `C` gira, `P` pausa e `Q` sai. Precisa de terminal interativo com pelo menos 23 linhas e 23 colunas e suporte a `curses`; o suporte pode variar no Android/Termux. Não usa rede nem cria arquivos.

#### `host_diagnostics` — diagnóstico de um único host

```sh
python legacy/host_diagnostics/host_diagnostics.py example.org --ports 80,443 --timeout 2
```

Resolve DNS, tenta DNS reverso e verifica somente as portas indicadas (máximo de oito). Uma conexão TCP aceita só prova que a porta aceitou a conexão; não prova que o serviço esteja saudável. Uma falha pode ser causada por firewall ou indisponibilidade temporária. Não examina faixas de IP nem faz fingerprinting. Use apenas em host próprio ou autorizado.

#### `local_toolbox` — utilitários para arquivos locais

```sh
python legacy/local_toolbox/local_toolbox.py hash arquivo.zip --algorithm sha256
python legacy/local_toolbox/local_toolbox.py stats notas.txt
python legacy/local_toolbox/local_toolbox.py tree pasta --depth 2
python legacy/local_toolbox/local_toolbox.py format-json entrada.json --output formatado.json
```

`hash` calcula um resumo criptográfico, mas não comprova a origem do arquivo. `stats` conta bytes, linhas e palavras de texto UTF-8. `tree` mostra uma árvore limitada pela profundidade. `format-json` valida e formata JSON; grava exatamente no caminho de `--output` e pode substituir o destino existente. Escolha um novo nome se quiser preservar o arquivo anterior.

#### `pulse_assistant` — clima atual e definições em inglês

```sh
python legacy/pulse_assistant/pulse_assistant.py weather 'São Paulo'
python legacy/pulse_assistant/pulse_assistant.py define resilient
```

`weather` mostra condições atuais fornecidas pelo Open-Meteo; não é previsão nem serviço de emergência. `define` consulta uma definição em inglês. O local ou a palavra são enviados aos serviços públicos correspondentes; não passe endereços privados nem textos sensíveis.

#### `secure_passphrase` — gerar senha ou frase-senha

```sh
python legacy/secure_passphrase/secure_passphrase.py --mode passphrase --words 6 --capitalize --digits
python legacy/secure_passphrase/secure_passphrase.py --mode password --length 32
```

Usa o gerador aleatório seguro do Python e imprime o resultado no terminal; não salva nem envia. A lista de palavras é pequena e a ferramenta não armazena nem preenche credenciais. Proteja a saída: evite capturas de tela, logs e terminais compartilhados; considere um gerenciador de senhas para contas de alto valor.

## Privacidade e segurança

- `lookay login` usa HTTPS para o serviço da Kayepad; o token local em `~/.config/lookay/session.json` é secreto.
- `site_watch`, `header_guard`, `pyroom` e `host_diagnostics` fazem requisições de rede aos alvos especificados.
- `dependency_lens` envia nomes e versões de pacotes ao PyPI e, com `--osv`, ao OSV.
- `solar_ledger` envia coordenadas e data ao Open-Meteo; `curiosity_terminal` envia a busca à Wikimedia; `pulse_assistant` envia o local ou a palavra ao serviço correspondente.
- `file_sift`, `backup_buddy`, `snippet_box`, `local_toolbox`, `git_scribe`, `env_doctor` e `secure_passphrase` operam localmente, salvo o acesso de rede opcional descrito acima.
- Antes de compartilhar saídas JSON, logs, backups ou capturas de tela, verifique se não contêm dados pessoais, URLs privadas ou segredos.

## Atualizar o código

Antes de atualizar, verifique suas alterações locais:

```sh
cd "$HOME/MIATSUNO-Lab"
git status
```

Se não houver mudanças locais importantes, atualize com:

```sh
git pull --ff-only
```

Após uma atualização, consulte `--help` novamente: opções e ferramentas podem evoluir. O projeto está em desenvolvimento; verifique os READMEs individuais no repositório MIATSUNO-Lab para detalhes atualizados.
