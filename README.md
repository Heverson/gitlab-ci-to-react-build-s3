<p  align="center">
<img  src="https://user-images.githubusercontent.com/729786/83407080-5d385580-a3e6-11ea-8abe-2c7ff847e050.png"  width="300">
</p>
<p  align="center">
<strong>Gitlab-CI to React build AWS S3M</strong> <br />
Enviando seu build do React para um bucket do S3 📥
</p>  

# 👣 Passo a Passo

<a  href="#1---crie-um-bucket-no-s3">1. Crie um bucket no S3</a>
<a  href="#2---crie-um-usuário-iam-que-permite-o-upload-para-o-bucket-s3-">2. Crie um usuário IAM que permite o upload para o bucket S3</a>
<a  href="#3---defina-a-configuração-no-ci-do-gitlab">3. Defina a configuração no CI do GitLab</a>
<a  href="#uma-observação-importante-">4. Rode o código e &#128591;</a>

## 1 - Crie um bucket no S3

Para configurar o S3, acesse o console de gerenciamento do S3, crie um novo *bucket*, digite o nome (por exemplo, seudominio.com.br) e a região. No final, deixe as configurações padrão.

<img  src="https://user-images.githubusercontent.com/729786/81438285-e390ad00-9142-11ea-83fe-2e39472335b8.png"  width="500"  height="160">

Depois disso, defina permissões para o acesso público do seu novo *bucket*. Dessa forma, você torna os arquivos do site acessíveis aos usuários pelo navegador.

<img  src="https://user-images.githubusercontent.com/729786/81438333-fc995e00-9142-11ea-8c8c-771d0d7d0011.jpg"  width="500"  height="260">

Agora vá para a guia *Properties* e selecione "*Static website hosting*". Marque a caixa *"Use this bucket to host a website"* e digite o caminho da página raiz (**index.html** por padrão) no campo *"Index Document"*. Além disso, preencha as informações necessárias no campo *"Error Document"*, se não tiver página de erro pronta, coloque o **index.html** novamente.

<img  src="https://user-images.githubusercontent.com/729786/81438393-1470e200-9143-11ea-999f-522dd5470c60.png"  width="550"  height="300">  

Agora coloque as permissões ao seu ***bucket S3*** para tornar seu site visível e acessível aos usuários. Vá para a guia *Permissions* e clique em *Bucket policy*. Insira o seguinte snippet de código no editor que aparece:

```
{
	"Version": "2012-10-17",
	"Statement": [
		{
		"Sid": "PublicReadForGetBucketObjects",
		"Effect": "Allow",
		"Principal": "*",
		"Action": "s3:GetObject",
		"Resource": "arn:aws:s3:::seubucket/*"
		}
	]
}
```

## 2 - Crie um usuário IAM que permite o upload para o bucket S3 🗳

  
Nesse estágio, você deve criar um usuário do IAM para acessar e fazer upload de dados para seu **bucket**. Para fazer isso, vá para o console de gerenciamento do **IAM** e clique em "Adicionar usuário" para criar uma nova política com o nome que você escolheu.

Depois disso, adicione o seguinte código. Não se esqueça de substituir o valor do campo "*Resource*" pelo nome que você criou. Assim, você permite que os usuários obtenham dados do seu **bucket**. 

O próximo passo é criar um novo usuário. Marque *Programmatic access* na seção *access type* e atribua-o à *policy* recém-criada.

Por fim, clique em "Create user". Haverá dois valores importantes: variáveis **AWS_ACCES_KEY_ID** e **AWS_SECRET_ACCESS_KEY**.

Se você fechar a página, você perderá acesso ao **AWS_SECRET_ACCESS_KEY**.

**É por isso que recomendamos que você anote a chave ou faça o download do arquivo .csv**.

## 3 - Defina a configuração no CI do GitLab

Agora você precisa iniciar o processo de implantação do seu projeto no *bucket S3*. Esse estágio supõe a configuração correta do GitLab CI. Faça login na sua conta do GitLab e navegue até o projeto. Clique em *Settings*, vá para a seção CI / CD e pressione o botão "*Variables*" no menu suspenso. Aqui, insira todas as variáveis necessárias, a saber:

- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- AWS_REGION
- S3_BUCKET_NAME

Depois disso, você precisa informar ao GitLab como seu site deve ser implantado no AWS S3. Isso pode ser feito adicionando o arquivo **.gitlab-ci.yml** ao diretório raiz do seu aplicativo. Simplificando, o *GitLab Runner* executa os cenários descritos neste arquivo.

No exemplo deixei

- build
- test
- deploy (Vamos focar nesse *stage*)
 

## Uma observação importante &#128588;

Execulte no projeto o comando para gerar o build

```yarn build ```

Tentei integrar a criação do build do react no .gitlab-ci.yml, mas estava gerando alguns erros 🐛, caso deseje fazer um pull request fique a vontado 😇!

Muitos tutoriais usavam NodeJS para rodar um package chamado **aws-cli**, más a biblioteca da AWS com NodeJS está descontinuada.

Por isso não vamos poder rodar o package da aws com **NodeJS**, más sim carregando uma imagem do **Python**.

```
deploy:
	image: "python:latest" # Usamos python porque ele trabalha com a biblioteca AWS Sdk
	stage: deploy
	artifacts:
	paths:
		- build
	before_script:
	- pip install awscli # instalando a SDK
	script:
	- aws s3 sync build s3://$S3_BUCKET_NAME/ # enviamos os arquivos da pasta build para o bucket no s3

```
> *Faça o upload desse projeto no seu repositório do GitLab*

Tendo o projeto no seu repositório no GitLab é só fazer o seu *commit*.
