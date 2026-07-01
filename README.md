# Lab 1 — Hospedagem de Site Estático com Amazon S3

Parte do projeto **InfraLab Cloud Foundations** — laboratório autodirigido de estudos práticos em AWS.

## Objetivo

Hospedar um site HTML estático utilizando exclusivamente o Amazon S3, sem servidor, sem EC2 — apenas o endpoint de website nativo do S3 respondendo às requisições HTTP.

## Serviços utilizados

- **Amazon S3** (Simple Storage Service)

## Arquitetura

```
Usuário (navegador)
        │
        │  HTTP GET
        ▼
Endpoint de Website S3
(http://vinicius-static-site-lab.s3-website-us-east-1.amazonaws.com)
        │
        ▼
  Bucket S3: vinicius-static-site-lab
  ├── index.html
  └── error.html
```

O S3, quando configurado com Static Website Hosting, se comporta como um servidor web simples: recebe requisições HTTP e retorna os objetos armazenados diretamente, sem necessidade de infraestrutura de computação (EC2, containers, etc).

## Passo a passo executado

### 1. Criação do bucket
Bucket criado com nome globalmente único: `vinicius-static-site-lab`, na região `us-east-1`.

### 2. Upload dos arquivos
Upload do `index.html` (e `error.html`) para a raiz do bucket.

### 3. Habilitação do Static Website Hosting
Em **Properties → Static website hosting**, habilitado com:
- Index document: `index.html`
- Error document: `error.html`

Isso gerou o endpoint público:
```
http://vinicius-static-site-lab.s3-website-us-east-1.amazonaws.com
```

### 4. Desativação do Block Public Access
Por padrão, a AWS bloqueia qualquer configuração pública em buckets S3 como camada de proteção. Foi necessário desativar manualmente essa trava em **Permissions → Block public access**, com confirmação explícita.

### 5. Aplicação da Bucket Policy
Política JSON aplicada em **Permissions → Bucket policy**, concedendo leitura pública (`s3:GetObject`) a todos os objetos do bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::vinicius-static-site-lab/*"
    }
  ]
}
```

### 6. Teste de acesso
Site testado com sucesso via navegador acessando o endpoint gerado.

## Funcionamento

O Amazon S3 é um serviço de armazenamento de objetos. Por padrão, todo bucket e todo objeto dentro dele são **privados**. Para servir arquivos como um site público, três coisas precisam acontecer em conjunto:

1. **Static Website Hosting** precisa estar habilitado, definindo qual arquivo é servido como página padrão (`index.html`) e qual é servido em caso de erro (`error.html`).
2. **Block Public Access** precisa estar desativado, pois é uma camada de segurança que impede qualquer configuração pública, mesmo que exista uma policy permitindo acesso.
3. **Bucket Policy** precisa conceder explicitamente a permissão de leitura (`s3:GetObject`) para o público (`Principal: "*"`).

Sem qualquer um desses três, o site não fica acessível — mesmo que os outros dois estejam corretos.

## Observação importante: HTTP vs HTTPS

O endpoint de Static Website Hosting do S3 (`*.s3-website-<região>.amazonaws.com`) **funciona apenas via HTTP**, sem certificado SSL/TLS nativo. Alguns navegadores e dispositivos forçam automaticamente o upgrade da URL para `https://`, o que resulta em falha de conexão — não por erro de configuração, mas porque esse protocolo não existe nesse tipo de endpoint.

Solução aplicada: acessar a URL digitando explicitamente `http://` no início, sem permitir que o navegador force o redirecionamento para HTTPS.

> Para servir o conteúdo via HTTPS, a abordagem correta seria colocar o Amazon CloudFront na frente do bucket S3 — tema de um lab futuro.

## Resultado

Site estático acessível publicamente via:
```
http://vinicius-static-site-lab.s3-website-us-east-1.amazonaws.com
```

## Prints

_(adicionar aqui os screenshots: criação do bucket, upload, static hosting habilitado, bucket policy salva, site funcionando no navegador)_

---

**Próximo lab:** Fase de Networking — VPC, Subnets e Security Groups.
