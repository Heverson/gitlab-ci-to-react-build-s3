# Gitlab-CI deploy React build AWS S3
Enviando seu build do React para um bucket do S3

# Passo a Passo

 1. Crie um bucket no S3
 2. Crie um usuário IAM que permite o upload para o bucket S3
 3. Defina a configuração no CI do GitLab
 4. Rode o código e &#128591;

## 1 - Crie um bucket no S3
Para configurar o S3, acesse o console de gerenciamento do S3, crie um novo *bucket*, digite o nome (por exemplo, seudominio.com.br) e a região. No final, deixe as configurações padrão.

Depois disso, defina permissões para o acesso público do seu novo *bucket*. Dessa forma, você torna os arquivos do site acessíveis aos usuários pelo navegador.

Agora vá para a guia *Properties* e selecione "*Static website hosting*". Marque a caixa *"Use this bucket to host a website"* e digite o caminho da página raiz (**index.html** por padrão) no campo *"Index Document"*. Além disso, preencha as informações necessárias no campo *"Error Document"*, se não tiver página de erro pronta, coloque o **index.html** novamente.

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
## 2 - Crie um usuário IAM que permite o upload para o bucket S3

Nesse estágio, você deve criar um usuário do IAM para acessar e fazer upload de dados para seu **bucket**. Para fazer isso, vá para o console de gerenciamento do **IAM** e  clique em "Adicionar usuário" para criar uma nova política com o nome que você escolheu.

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

