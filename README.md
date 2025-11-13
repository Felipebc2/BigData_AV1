# 📊 AV1_BIGDATA

Considerando os fundamentos teóricos e práticas de laboratório ensinados na matéria de Big Data lecionada por [Klayton Castro](https://github.com/klaytoncastro), exploramos várias ferramentas e conceitos de Big Data, como exemplo Docker, MongoDB, Cassandra, Shell, NoSQL, entre outras... Com base na configuração original e exemplos do repositório [Idp-bigdata](https://github.com/klaytoncastro/idp-bigdata) esse projeto foi criado para ser avaliado na matéria do professor como AV1 (Atividade Avaliativa 1).

# 🐍 Ferramentas e Linguagens Utilizadas
<img src="https://skillicons.dev/icons?i=py,cassandra,docker,mongodb,linux,powershell,vim" />

## ✅ O que foi Desenvolvido?
- Criação do Ambiente do Container com Docker
- Implementação de Pontes de Conexão para interligar os sistemas com .yaml
- Subida de ambientes com Docker Compose (MongoDB, Cassandra, Jupyter).
- Manipulação de dados:
  - CRUD completo com MongoDB e Cassandra.
  - Operações agregadas com MongoDB.
- Integração entre Jupyter e bancos NoSQL com `pymongo` e `cassandra-driver`.
- Tratamento dos Dados do INEP
- Análise exploratória de dados do Censo da Educação Superior 2022:
  - Número de instituições por região e estado.
  - Proporção de docentes por gênero.
  - Visualização com gráficos usando Matplotlib e Seaborn.
 
## 📂 Estrutura do Projeto

```
BigData_AV1/
│
├── docker-compose.yml             # Arquivo unificado para Jupyter, MongoDB e Cassandra
├── permissions.sh                 # Script para fornecer permissão
├── wair-for-it.sh                 # Script para funcionamento do MongoDB e Cassandra
├── config/                        # Configuração do Jupyter (jupyter_server_config.json)
├── notebooks/
│   └── AnaliseDeDados.ipynb      # Notebook da Analise de Dados do Dataset Tratado
│   └── Cassandra.ipynb           # Notebook com operações e análise Cassandra
│   └── Mongo.ipynb               # Notebook com operações e análise MongoDB
```

## 🚀 Como Executar o Projeto

### Pré-requisitos

- Docker e Docker Compose instalados
- Python 3 com bibliotecas:
  - `pymongo`
  - `cassandra-driver`
  - `pandas`
  - `matplotlib`
  - `seaborn`
  - Dataset INEP tratado

#### Como Obter e tratar o Dataset do INEP?

a) Em seu terminal, baixe e descompacte o arquivo do dataset utilizando os comandos abaixo: 

```bash
cd /opt/idp-bigdata/mongodb/datasets
wget https://download.inep.gov.br/microdados/microdados_censo_da_educacao_superior_2022.zip --no-check-certificate
unzip microdados_censo_da_educacao_superior_2022.zip
```
b) Como precisaremos apenas dos arquivos CSVs (Comma Separated Values) com as IES e Cursos por elas disponibilizados, organize a subpasta `datasets` executando os comandos abaixo para manter apenas a estrutura de pastas e dados estritamente necessários. 

```bash
rm microdados_censo_da_educacao_superior_2022.zip && mv microdados_educaЗ╞o_superior_2022 inep
mv inep/dados/*.CSV inep
rm -rf inep/dados && rm -rf inep/Anexos && rm -rf inep/leia-me
```

c) Precisaremos realizar a remoção e substituição de caracteres indesejados para viabilizar a correta importação do dataset. Vá até a subpasta do dataset (`cd /opt/idp-bigdata/mongodb/inep`) e execute o comando abaixo:

```bash
sed 's/\"//g; s/;/,/g' MICRODADOS_ED_SUP_IES_2022.CSV > MICRODADOS_ED_SUP_IES_2022_corrigido.csv
```

d) Nos sistemas Linux, o comando `file` é útil para mostrar informações sobre a tipagem do arquivo. A opção `-i` solicita a apresentação do padrão de codificação. Ambos utilizam o padrão ISO 8859-1. Seguem os comandos para verificar a codificação aplicada aos arquivos:

```bash
file -i MICRODADOS_ED_SUP_IES_2022_corrigido.csv
file -i MICRODADOS_CADASTRO_CURSOS_2022.CSV
```
e) Nos sistemas Linux, o utilitário `iconv` é utilizado para converter a codificação de caracteres de determinado arquivo. O trecho `-f ISO-8859-1` indica a codificação original do arquivo e o trecho `-t UTF-8` indica a codificação desejada (conversão). Assim, com o comando abaixo podemos converter os arquivos do formato ISO 8859-1 para UTF-8 requerido por nosso ambiente MongoDB: 

```bash
iconv -f ISO-8859-1 -t UTF-8 MICRODADOS_ED_SUP_IES_2022_corrigido.csv > MICRODADOS_ED_SUP_IES_2022_corrigido_UTF8.csv
```
#### Normalização de Texto

```bash
sed -i 'y/áàãâäéèêëíìîïóòõôöúùûüçñÁÀÃÂÄÉÈÊËÍÌÎÏÓÒÕÔÖÚÙÛÜÇÑ/aaaaaeeeeiiiiooooouuuucnAAAAAEEEEIIIIOOOOOUUUUCN/' MICRODADOS_ED_SUP_IES_2022_corrigido_UTF8.csv
```
#### Importação para o MongoDB

```bash
docker exec -it mongo_service mongoimport --db inep --collection ies --type csv --file /datasets/inep/MICRODADOS_ED_SUP_IES_2022_corrigido_UTF8.csv --headerline --ignoreBlanks --username root --password mongo --authenticationDatabase admin
```
a) Agora vamos repetir todo o processo de preparação, limpeza e importação para o arquivo que irá alimentar a collection `cursos`: 

```bash
sed 's/\"//g; s/;/,/g' MICRODADOS_CADASTRO_CURSOS_2022.CSV > MICRODADOS_CADASTRO_CURSOS_2022_corrigido.CSV

iconv -f ISO-8859-1 -t UTF-8 MICRODADOS_CADASTRO_CURSOS_2022_corrigido.CSV > MICRODADOS_CADASTRO_CURSOS_2022_corrigido_UTF8.CSV

sed -i 'y/áàãâäéèêëíìîïóòõôöúùûüçñÁÀÃÂÄÉÈÊËÍÌÎÏÓÒÕÔÖÚÙÛÜÇÑ/aaaaaeeeeiiiiooooouuuucnAAAAAEEEEIIIIOOOOOUUUUCN/' MICRODADOS_CADASTRO_CURSOS_2022_corrigido_UTF8.CSV

docker exec -it mongo_service mongoimport --db inep --collection cursos --type csv --file /datasets/inep/MICRODADOS_CADASTRO_CURSOS_2022_corrigido_UTF8.CSV --headerline --ignoreBlanks --username root --password mongo --authenticationDatabase admin
```

---

- ## 📁 Referências Base

- Jupyter: https://github.com/klaytoncastro/idp-bigdata/tree/main/jupyter  
- MongoDB: https://github.com/klaytoncastro/idp-bigdata/tree/main/mongodb  
- Cassandra: https://github.com/klaytoncastro/idp-bigdata/tree/main/cassandra  
