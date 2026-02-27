# Nginx & Certbot Docker Boilerplate

Um template inicial (boilerplate) para criar aplicações web com Docker Compose, estruturado para utilizar **Nginx** como proxy reverso e **Certbot** para geração e renovação automática de certificados SSL (HTTPS).

## 🚀 Como usar

1. **Utilize como base:**
   Clone este repositório ou utilize-o como template para iniciar a infraestrutura do seu projeto.

2. **Configure os domínios da sua aplicação:**
   Dentro da pasta `nginx`, faça uma cópia do arquivo de exemplo de ambiente:

   ```bash
   cp nginx/domains.env.example nginx/domains.env
   ```

   Edite o arquivo `nginx/domains.env` e altere os valores para os seus domínios reais (exemplo: seu domínio principal e o domínio da API).

3. **Integre os seus serviços:**
   O arquivo `docker-compose.yaml` define o proxy, o gerador de certificados e a rede isolada `app-network`. Para que suas aplicações fiquem acessíveis, você pode adicionar seus próprios serviços (containers) no mesmo arquivo, ou utilizar um `docker-compose.override.yaml`.

   **Exemplo de integração no `docker-compose.yaml`:**

   ```yaml
   services:
     # ... (serviços nginx e certbot originais) ...

     web:
       image: minha-imagem-frontend
       networks:
         - app-network

     api:
       image: minha-imagem-backend
       networks:
         - app-network
   ```

   _Nota:_ O arquivo `nginx/nginx.conf.template` vem configurado por padrão para rotear o tráfego do `APP_DOMAIN` para o host `http://web:3000` e do `API_DOMAIN` para o host `http://api:3000`. Você pode ajustar nomes de hosts e portas no template conforme necessário.

4. **Gerando Certificados SSL (Primeira Execução):**
   Como o Nginx está configurado para tentar carregar certificados `fullchain.pem` e `privkey.pem` logo na inicialização, ele falhará se os certificados ainda não existirem.
   Para gerar na primeira vez:
   - Comente temporariamente os blocos HTTPS (`listen 443 ssl;` e as diretivas de certificado) dentro de `nginx/nginx.conf.template`.
   - Suba o Nginx com o Nginx servindo somente na porta 80:
     ```bash
     docker compose up -d nginx
     ```
   - Gere os certificados rodando um container efêmero do Certbot contra o seu Nginx que já está rodando (substitua os domínios):
     ```bash
     docker compose run --rm certbot certonly --webroot --webroot-path /var/www/certbot/ -d meudominio.com -d api.meudominio.com
     ```
   - Após o sucesso na geração, remova os comentários da configuração HTTPS do Nginx no template e reinicie:
     ```bash
     docker compose down && docker compose up -d
     ```

## 📂 Estrutura de Pastas

- `docker-compose.yaml`: A definição principal da infraestrutura.
- `nginx/`: Templates para as regras de roteamento. O `docker-compose` utiliza o comando `envsubst` para substituir as variáveis definidas em `domains.env` diretamente dentro dos arquivos de `.template`.
- `certbot/`: Pastas montadas como volume que garantem que os certificados gerados no container do Certbot sejam acessíveis e persistidos para o Nginx.

## 💡 Dicas de Manutenção

### Testando a Configuração do Nginx

Sempre que mexemos no NGINX via Docker, um erro comum é o container não subir por erro de sintaxe. Antes de reiniciar o container após uma mudança no template, você pode validar a sintaxe gerada dentro do container:

```bash
docker exec -it nginx-proxy nginx -t
```

Se a saída for `syntax is ok` e `test is successful`, é seguro reiniciar o Nginx:

```bash
docker restart nginx-proxy
```

## 👨‍💻 Autor

![Victor Lis Bronzo](https://gitassets.victorlisbronzo.me/api/card/cmm0ns5e0000p0iprt7eonzqy?v=5jf40s)
