# Aula básica de inicializar um projeto DJANGO com DOCKER e POSTGRESQL

1 - Primeiro passo é criar a pasta do projeto

2 - Instalar o ambiente virtual e inicializar
- python -m venv venv
- venv/Scripts/activate
- pip install django

O Passo 3 e 4 a seguir podem serem feitos em qualquer ordem, anyway.

3 - Como estamos mexendo com Docker para deploy futuramente, etc. 
Criamos o arquivo requirements para deixar nossas dependências para instalar
posteriormente, que serão utilizadas ao rodar o docker-compose up
- requirements.txt criado com o conteúdo:
- DJANGO==3.1.2 (*VERSION*)
- psycopg2-binary==2.8.6 (*VERSION*) 

4 - Podemos agora ou antes do passo 3 startar nosso projeto django
- django-admin startproject auladocker . <- importante para não criar outra pasta, pois ja criamos a nossa.
- Após criar nosso projeto, podemos testar se está tudo certo até agora:
- python manage.py runserver ( vai estar tudo ok )

5 - Mexendo agora com os 2 arquivos do docker, criar Dockerfile e docker-compose.yml
- Em Dockerfile, sempre se iniciar com o FROM ( imagem base da construção da nossa imagem ( no banco))
- Agora duas variáveis de ambiente, PYTHONDONTWRYTEBYTECODE 1 ( que serve para não gerar arquivos .pyc )
- E a outra PYTHONUNBUFERRED 1 ( as mensagens de logs não ficarão armazenadas em buffer e sim entregues imediatamente)
- Agora o nosso ambiente de trabalho : WORKDIR /code ( onde nosso banco vai jogar os códigos )
- Agora o COPY requirements.txt . ( vai jogar tudo que tiver no requirements.txt para dentro do diretorio de trabalho ( WORKDIR /code))
- Agora o RUN, que ele vai rodar o pip install -r requirements ( instalar as dependencias do arquivo requirements)
- Agora o COPY . . ( copiar tudo do direitorio atual (.) para o (.) /code ( WORKDIR ))

6 - Mexendo agora com o docker-compose.yml
- Começamos o docker-compose com o version= "3.8" ( que colocamos a versão do compose file format )
- Agora definimos os services, que serão 2 'web' e 'db'
- o web começa com o nosso build: . que diz que o no docker file esta no direitorio atual ( . )
- Apos o build, agora executamos no web o command: python manage.py runserver 0.0.0.0:8000 ( inicia o servidor de dev do django)
- Apos o command é interessante por o volumes: - .:/code ( sincroniza os arquivos locais com o container ) pois estaremos constantemente editando no local nosso projeto.
- Apos o volumes colocamos as ports: - 8000:8000 ( fazer o expose das portas padrao do django)
- Apos o ports colocamos o depends_on : - db ( que diz que o serviço web depende do db )
- Apos isso, terminamos a parte do WEB e vamos para o serviço db
- No db começamos colocando a imagem do banco : image: postgres:13
- Agora colocamos nossas variaveis do environment: - POSTGRES-USER=postgres e POSTGRES-PASSWORD=postgres
- Apos isso colocamos os volumes ( IMPORTANTE PARA OS DADOS PERSISTIREM )
- volumes: postgres_data:/var/lib/postgresql/data/
- Serviço db finalizado, colocamos fora dos serviços os volumes 
- volumes : postgres_data:

7 - Vamos alterar no settings o banco para postgresql
- DATABASES {
- 'default': {
- 'ENGINE' : 'django.db.backends.postgresql', 
- 'NAME' : 'postgres',
- 'USER' : 'postgres',
- 'PASSWORD' : 'postgres',
- 'HOST' : 'db',
- 'PORT' : 5432, }}

8 - Com isso finalizamos nosso setup docker, agora vamos rodar para ver se está tudo correto
- Comando que utilizaremos para rodar o docker:
- docker-compose (comandos...).
- inicialmente começamos com docker-compose up, para levantar nosso servidor docker.
- Nele vai fazer 7 etapas descritas nos nossos docker files
- pegar a imagem do python .. pegar as variaveis de ambiente,  copiar os requirements, dar run etc ..
- Se formos no endereço pra ver se está tudo certo, estará lá 127.0.0.1:8000
- Podemos rodar nosso migrate, como estamos pelo docker rodamos assim :
- docker-compose exec web python manage.py migrate
- Podemos rodar o createsuperuser e acessar a parte de adm do site.. etc
FIM
