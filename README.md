# Coletor para o Instagram

Coleta perfis com utilizando [instaloader](https://instaloader.github.io/) e armazena localmente

### Uso

Faz download de mídias e comentários de uma lista de perfis, dada uma data de início para a coleta

Clone este repositório com o comando:

```
git clone https://github.com/MPMG-DCC-UFMG/C02-Instagram
```

Execute com:

```
python3 -B init_crawler.py --json entrada.json
```

A entrada recebida pelo programa para a coleta de posts por perfis tem o formato:

```
{
  "users": ["anvisaoficial"],
  "min_date": "2020,6,18",
  "sleep_time": 1,
  "users_to_download_media": ["anvisaoficial"],
  "max_comments": 1,
  "users_to_download_followers": ["minsaude", "anvisaoficial"],
  "followers_max": 14,
  "user": "XXXXXXX",
  "passwd": "XXXXXXX",
  "crawler": "instagram",
  "download_hashtags": false,
  "hashtags_list": [],
  "hashtags_max": null
}
```

Já para a coleta de posts por hashtags (note *download_hashtags*, *hashtags_list* e *hashtags_max*):

```
{
  "users": [],
  "min_date": "",
  "sleep_time": null,
  "users_to_download_media": [],
  "max_comments": null,
  "users_to_download_followers": [],
  "followers_max": null,
  "user": "",
  "passwd": "",
  "crawler": "instagram",
  "download_hashtags": true,
  "hashtags_list": ["testeste", "asasasasas"],
  "hashtags_max": 2
}
```

Os argumentos são:

- `users`: Lista de perfis que serão coletados, incluindo informações sobre posts, perfis e comentários
- `min_date`: Data de início da coleta. O default é ontem, caso uma data não seja específicada. O formato da data deve ser `YYYY,MM,DD`, sem zeros à esquerda. Exemplo: 20/03/2020, deve-se _remover_ os zeros à esquerda, da seguinte forma: 2020,3,20
- `sleep_time`: Tempo (em segundos) de espera entre coleta de perfis. Caso não for utlizar, definir como `0`.
- `users_to_download_media`: lista de perfis que terão suas mídias coletadas. Isso significa fazer download dos vídeos e imagens postados na timeline do perfil, dentro do período especificado. Note que este campo deve conter um subconjunto do campo `users`, pois não é possível realizar o download de mídias de perfis que não foram coletados.
- `max_comments`: máximo de comentários _por post_ que devem ser coletados
- `users_to_download_followers` : Lista de perfis cujas listas de seguidores devem ser coletadas.
- `followers_max`: Número máximo de seguidores que serão coletados _por perfil_. Caso queira coletar a lista completa de seguidores, utilizar o valor `null` neste campo.
- `user`: Nome de usuário da conta ativa do instagram necessária para realizar o credenciamento necessário para realizar download da lista de seguidores
- `passwd`: Senha da conta ativa do instagram necessária para realizar o credenciamento necessário para realizar download da lista de seguidores
- `download_hashtags`: Deve ser ou "true", caso queira coletar posts por hashtags ou "false" caso queira coletar posts por perfis.
- `hashtags_list`: Lista de hashtags que devem ter seus posts coletados,
- `hashtags_max`: Número máximo de posts que devem ser coletados _por hashtag_. Caso queria coletar todos os posts disponíveis, utilizar este campo com o valor `null`.
- `crawler`: deve sempre ter "instagram" como argumento, pois usa um módulo genérico que faz download de mais de uma rede social.

Uma outra opção é passar o arquivo json diretamente na chamado do coletor:

```
python3 -B init_crawler.py "{'users': ['minsaude'], <...>}"
```

### Pacotes

Python3 com os seguintes pacotes nas versões especificadas

Name: instaloader
Version: 4.4.4
Name: timeout-decorator
Version: 0.4.1

```
pip3 install -r requirements.txt
```

Se utilizar o Anaconda, pode ser que haja algum problema com a instalação do timeout-decorator. Melhor usar o PIP para instalar os pacotes

### Observações

- Lembrar também de dar permisão aos arquivos: chmod +x script
- _Importante_: só é possível iniciar uma coleta por vez em uma mesma pasta com os arquivos fonte. Isto é necessário para garantir que as manipulações e movimentações de arquivos e pastas, necessárias para gerar as saídas corretamente, tenham integridade preservada. Para contornar isso, é possível alinhar as coletas para executar em sequência em uma mesma pasta, ou então executar coletas paralalemante em pastas diferentes. Este último deve ser evitado, para evitar queries excessivas ao Instagram.
- Não executar o Instagram em navegador ao mesmo tempo que realiza 

### Utilização com Docker

Para que a instalação e execução dos scripts de coleta sejam feitas de forma facilitada, optamos por utilizar a plataforma Docker para encapsular o código e suas dependências.

#### Instalação

Primeiramente, faça a instalação do Docker seguindo os seguintes [passos](https://docs.docker.com/engine/install/), e instale também o Docker Compose seguindo estes [passos](https://docs.docker.com/compose/install/).

Com o docker instalado, clone este repositório para sua máquina. Dentro da pasta criada, rode o comando `sudo docker-compose build` para que as imagens necessárias sejam criadas.

Em seguida, como teste, rode o comando `sudo docker-compose run --rm instagram-collector` para iniciar uma coleta (por padrão, fará uma coleta utilizando as configurações localizadas no arquivo [coleta.json](https://github.com/MPMG-DCC-UFMG/C02-Instagram/blob/master/input/coleta.json)).

Mais informações sobre a necessidade de executar os comandos com privilégios `sudo` podem ser encontradas [aqui](https://docs.docker.com/engine/install/linux-postinstall/).


#### Execução

Com a instalação finalizada, os scripts do coletor estarão localizados em um container que se chama ```instagram-collector```.

A utilização do coletor com o Docker se diferencia um pouco da utilização normal de scripts python. Primeiramente, como os scripts serão executados dentro do container, os comandos de execução serão sempre acrescidos do comando que chama o container em que eles estão contidos. Por exemplo, em uma situação normal em que a execução do script do coletor pode ser feita pelo comando ```python init_crawler.py```, para que a mesma coisa seja feita seja feita dentro do container ```instagram-collector```, o comando seria ```docker-compose run instagram-collector python init_crawler.py```.

Além disso, como os scripts são executados dentro de um container que se comporta como uma máquina virtual, eles só conseguem acessar pastas que estão dentro desta “máquina” (Isto inclui os arquivos de entrada e saída). Para fazer com que o código tenha acesso a pastas do sistema operacional do usuário, o arquivo ```docker-compose.yml``` define um espelhamento de pastas na seção ‘volumes’. Cada espelhamento está no formato ```/pasta/do/sistema/operacional/local:/caminho/para/pasta/no/container```, que define que a pasta local no Sistema Operacional será acessível dentro do container no caminho escolhido. Por padrão, está somente definido um volume que espelha a pasta raíz do repositório para a pasta raíz do container (a pasta ```/app```), fazendo com que seja possível o script acessar todo o conteúdo do repositório.


## Scripts

### init_crawler.py

Inicializa a coleta, realizando a leitura e interpretação da entrada e disparando os comandos de coleta de perfis, posts, comentários e mídias

### run_crawl.sh

Dispara a coleta de perfis, posts e comentários

### 1_download.sh

Este script lê um usuário por vez e passa como entrada para o instaloader, junto com os demais parâmetros especificados. O instaloader gera um log que é salvo na pasta /tmp/. Esse log é apresentado no terminal ao executar o código.

### download_comments.py

Utiliza os arquivos .xz para coletar os códigos identificadores de mídias/posts. Em seguida, utliza a interface do instaloader para coletar todos os comentários daquele post. Caso um comentário tenha respostas, coleta essa informação
deixando explícito qual o comentário pai e qual o filho.

### create_archives.py

Utiliza os arquivos de comentário e posts gerados para cada usuário. Apenas concatena
os comentários e informações sobre os posts obtidos pelo instaloader em dois arquivos
únicos para cada coleta (comments.json e medias.json).

### download_medias.py

Define uma classe que itera pelos posts, faz a requisição de cada uma das mídias (fotos ou vídeos) e salva dentro da pasta _images_ do _archives_ correspondente.

### followers.py

Define uma classe que realiza a coleta da lista de seguidores de perfis especificados na entrada.

### hashtags.py

Define uma classe para realizar coleta de posts que contenham uma hashtag

### utils.py

Arquivo de funções auxiliares: extract_files é responsável por extrair os arquivos comprimidos obtidos da coleta.

## Classes

Algumas classes foram implementadas, suas documentações seguem abaixo:

### Coletor

    Classe para inicializar o coletor. Realiza a leitura da entrada via
    argumento de linha de comando, e passa para os módulos que coletam
    as informações de cada perfil, assim como o download das mídias dos
    perfis selecionados.

    Atributos
    -----------
    input_json : str
        Nome do arquivo .json lido como entrada via linha de comando

    Métodos
    -----------
    Coletor():
        Construtor. Inicializa o objeto.

    init_crawler()
        Função que inicializa o crawler, lendo o arquivo .json de entrada
        e fazendo a chamada das funções que inicializam a coleta dos perfis
        e das mídias.

### DownloadComments

    Classe para coletar comentários de posts do instagram. Utiliza os posts
    coletados pela interface de linha de comando do Instaloader para isso

    Atributos
    ---------
    max_comments : int
        máximo de comentários *por post* que devem ser coletados

    input_dir : str
        nome da pasta em que se encontram os dados dos perfis coletados

    Métodos
    ---------
    DownloadComments(max_comments, input_dir):
        Inicializa o objeto

        Parâmetros
        ---------
        max_comments : int
            máximo de comentários *por post* que devem ser coletados

        input_dir : str
            nome da pasta em que se encontram os dados dos perfis coletados

    download_comments()
        Função que itera sobre as pastas dos perfis coletados, obtêm os códigos
        de posts de cada uma e dispara a coleta dos comentários para cada post

### CreateArchives

    Classe que cria arquivos e pastas que irão compilar informações
    de mídias, perfis, posts e comentários coletados pelo Instaloader

    Atributos
    ---------
    INPUT_DIR : str
        nome da pasta onde são armazenados os arquivos de entrada
    OUTPUT_DIR : str
        nome da pasta onde serão armazenados os arquivos de saída
    INPUT_ARCHIVE_COMMENTS : str
        nome do arquivo que irá compilar os comentários de todos os posts
    TIME : str
        timestamp utilizada para identificar cada pasta de coleta


    Métodos
    ---------
    CreateArchives(INPUT_DIR, OUTPUT_DIR, INPUT_ARCHIVE_COMMENTS):
        Construtor, inicializa o objeto

        Parâmetros
        ---------
        INPUT_DIR : str
            nome da pasta onde são armazenados os arquivos de entrada
        OUTPUT_DIR : str
            nome da pasta onde serão armazenados os arquivos de saída
        INPUT_ARCHIVE_COMMENTS : str
            nome do arquivo que irá compilar os comentários de todos os posts
        TIME : str
            timestamp utilizada para identificar cada pasta de coleta

    create_archives():
        Cria pastas e arquivos de saída, faz chamadas para
        funções que agregam comentários, posts, informações
        de perfis

### DownloadFollowers

    Classe para realizar o download de seguidores de uma lista
    de perfis especificada

    Atributos
    ---------
    self.input_json_data = self._get_input_json(filename)
        self.users_list = self._get_users_list()
        self.followers_max = self._get_followers_max()
        self.credentials = {}
        self.credentials["user"] = self._get_user()
        self.credentials["passwd"] = self._get_passwd()
        self.path = self._get_path()
    input_json_data : dict
        Dicionário que armazena o .json de entrada
    followers_max : int
        Número máximo de seguidores que devem ser baixados por perfil
    credentials : dict
        Dicionário que armazena nome de usuário e senha do conta ativa
        do instagram necessária para realizar a coleta de seguidores
    path : str
        Caminho para a pasta onde serão armazenados os arquivos com a lista de seguidores

    Métodos
    -------
    download_followers()
        Itera sobre a lista de perfis que devem ter a lista de seguidores baixada,
        realiza o download e armazena na pasta da coleta.

### download_medias

    Classe para realizar o download das mídias de posts de Twitter
    ou Instagram coletados via os coletores desenvolvidos para o
    projeto do Ministério Público de Minas Gerais.

    Atributos
    ---------
    folder : str
        Nome da pasta padrão da coleta do Instagram.
    path : str
        Nome do diretório de saída para o download das mídias.
    data : list de dict
        Lista dos posts coletados. Cada post é um dict contendo
        variadas informações (texto, data, url da mídia etc).
    users : list de str
        Lista de usuários cujas mídias devem ser baixadas. Caso seja
        None, todos os usuários devem ser considerados.
    is_twitter : bool
        Verdadeiro caso seja uma coleta de Twitter, falso caso seja
        de Instagram.

    Métodos
    -------
    download(verbose)
        Função que itera pelos posts e chama as funções
        correspondente ao tipo de post (instagram ou twitter)
        para baixar as mídias (se pertencerem a um dos usuários
        pré-definidos).

### DownloadHashtags

    Classe para inicializar o coletor. Realiza a leitura da entrada via
    argumento de linha de comando, e passa para os módulos que coletam
    as informações de cada perfil, assim como o download das mídias dos
    perfis selecionados.

    Atributos
    -----------
    input_json : dict
         Dicionário com as informações da entrada
    hashtags_list : list
         Lista de de hashtags que devem ser coletadas
    hashtags_max : int
         Número máximo de posts por hashtag que devem ser coletados
    time : int
         Timestamp do horário da coleta. Importante para gerar as
         saídas nas pastas corretas


    Métodos
    -----------
    download_hashtags()
        Método que realiza o download de hashtags especificadas na entrada

## Arquivos

### Pastas

O código cria uma pasta chamada "data", com uma subpasta "archives:

Dentro de archives, estão as pastas relativas a cada coleta iniciada, nomeadas com o timestamp do fim da coleta em [Unix time](https://en.wikipedia.org/wiki/Unix_time), desprezando os milisegundos. Dessa forma, é possível recuperar a ordem em que as coletas foram feitas, importante para organizar os arquivos coletados.

O coletor permite, ou a coleta de perfis, posts e comentários, ou a coleta de hashtags (palavras-chave):

#### Posts, perfis, comentários, seguidores e mídias:

Dentro de uma pasta de coleta, temos arquivos sobre os comentários (comments.json) e mídias (medias.json) de toda a coleta, compilados. Além disso, temos as pastas:

- Pasta staging: Separa os perfis em pastas, com um json específico para para os comentários daquele perfil, assim como um json para cada post coletado (o nome do arquivo corresponde ao timestamp de postagem)
- Pasta images: Imagens e vídeos coletados para os perfis especificados no campo "users_to_download_media" da entrada
- Pasta followers: Lista de seguidores de cada perfil especificados no campo "users_to_download_followers" da entrada

Dentro da pasta `/staging/` temos as pastas salvas para cada usuário coletado.
Dentro da pasta de um usuário (ex: `/staging/minsaude`), são armazenadas algumas informações: para cada usuário, baixa-se a foto de perfil, comentários para os posts do período especificado, id dada ao usuário, arquivo com informações sobre o usuário coletado e sobre as mídias dos posts coletados.

**Arquivo Id**: Contêm a id dada aquele perfil na coleta.

**Arquivos de Mídias**: Cada arquivo tem como título a timestamp de postagem do post coletado. Compreende informações sobre as mídias, bio do dono do post e algumas informações adicionais.

**Arquivo de texto principal do post**: Para um mesmo post, seu arquivo de mídias e seu arquivo de texto principal do post possuem a mesma nomenclatura: timestamp de postagem. No caso do texto principal do post, este é armazenado em um arquivo .txt

**Arquivo de Comentários**: Tem formato `comments_$USERNAME.json` e contêm todos os comentários dos posts coletados. Cada comentário tem os campos:

- text: texto do comentário
- created_time: timestamp da criação do comentário
- created_time_str: horário da criação em formato string
- media_code: identificador que o instagram usa em suas urls ex: www.instagram.com/p/B-F_9kMneA4/
- id: id gerado para o comentário
- owner_username: username do usuário que fez o comentário
- owner_id: id gerada para o dono do comentário
- tags: tags que o usuário usou no comentário
- mentioned_usernames: usernames que o dono do comentário citou
- parent_comment_id: se o comentário é uma resposta a outro, identifica qual o comentário original utilizando sua ID

**Arquivo de Informações do Perfil**: Tem formato `$USERNAME_$ID.json` e tem informações gerais sobre o perfil coletado.

#### Hashtags (Palavras-chave)

Dentro de uma pasta de coleta de hashtags, são criadas pastas para cada hashtag coletadas. Dentro da pasta de uma hashtag (ex: 'tbt'), temos arquivos com informações dos posts, as mídias de cada um, além de arquivo separado com o texto principal de cada postagem.
