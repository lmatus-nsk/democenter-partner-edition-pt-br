# Demo Center Netskope — Partner Edition

> **Edição em português (pt-BR).** Esta é a tradução para português do Brasil do portal (branch `claude/changes`). A edição original em espanhol continua na branch `main`; o código, as chaves de i18n, as rotas e as URLs são idênticos.

Portal web de demos da Netskope (LATAM), em **português do Brasil** nesta edição. É HTML/PHP estático: não há banco de dados, Composer nem Node em runtime.

A interface é **responsiva**: se adapta a laptop, tablet e celular. Para demos com clientes recomenda-se laptop em 1920×1080; o portal continua utilizável em telas menores.

Esta edição de partner:

- Somente um idioma (sem seletor de idioma).
- Borderbot / AWS Bedrock **desabilitado** (`api/demo-chat.php` responde 404).
- Seção Agentic / MCP **oculta** no HTML.
- Não inclui credenciais da AWS nem o arquivo `/etc/demo-chat.env`.

Uso previsto: copiar esta árvore para o document root de um servidor web e servir `index.php`.

---

## 1. Requisitos de hardware

O site é leve (~8 MB). Os mínimos abaixo deixam folga para o SO, logs, atualizações e demos em paralelo.

| Recurso | Mínimo | Ideal |
|---|---|---|
| CPU | 1 vCPU | 2 vCPU |
| RAM | 2 GB | 4 GB |
| Disco | 40 GB | 120 GB |
| Rede | 100 Mbps | 1 Gbps |
| Resolução da demo | 1366×768 | 1920×1080 (laptop do SE; o site é responsivo) |

Não é necessário GPU, fila de workers nem disco SSD de alto IOPS.

**Lab RDP / cadeiras Cursor (opcional, não vem neste repositório).** O portal original consulta um host Windows interno (`10.1.10.203:18080`) e links RDP em `10.1.10.x`. Isso é infraestrutura do lab LATAM e não é instalado com este código. Se o partner não tiver esse lab, o quadro de cadeiras mostra um erro controlado e o restante do site funciona.

---

## 2. Requisitos de software (sistema)

Testado com PHP 8.x. O código usa tipos escalares (`string`, `int`, `void`) → **PHP 7.4 ou superior**. Recomendado **PHP 8.1+**.

| Componente | Versão | Para quê |
|---|---|---|
| Sistema operacional | Ubuntu 22.04 / 24.04 LTS (ou equivalente RHEL 9) | Servidor web |
| nginx | 1.18+ | HTTP(S). Apache 2.4 também serve |
| PHP-FPM | 8.1+ | Executar `*.php` |
| Git | 2.x | Clonar este repositório |
| Certificado TLS | Let's Encrypt ou outro | HTTPS em demos com cliente |
| Navegador | Chrome / Edge recentes | Conduzir a demo |

**Não é necessário:** MySQL/PostgreSQL, Redis, Node.js, Python, Composer, npm, Docker, AWS CLI, Bedrock nem Java.

### Pacotes a instalar (Ubuntu / Debian)

```bash
sudo apt update
sudo apt install -y nginx php-fpm php-cli git curl unzip
```

No Ubuntu 24.04 isso costuma trazer o PHP 8.3. Confira:

```bash
php -v
php-fpm -v
nginx -v
```

Extensões PHP usadas (vêm no pacote padrão):

- `json`
- `fileinfo` (imagens)
- wrapper HTTP do `file_get_contents` (`allow_url_fopen = On` no `php.ini`, valor padrão)

Opcional, somente se quiserem HTTPS com Let's Encrypt:

```bash
sudo apt install -y certbot python3-certbot-nginx
```

### Rede e firewall

| Porta | Direção | Uso |
|---|---|---|
| 80 / 443 | Entrada | Site público |
| 22 | Entrada (admin) | SSH |
| Saída 443 | Para a Internet | CDNs (veja abaixo) |

Várias páginas carregam CSS/JS da Internet. O servidor **e** o laptop da demo precisam de saída HTTPS para:

- `cdn.tailwindcss.com`
- `unpkg.com`
- `cdn.jsdelivr.net` (Chart.js nos relatórios)
- `fonts.googleapis.com` / `fonts.gstatic.com`

Se o partner bloquear CDNs, essas páginas aparecerão sem estilos ou sem gráficos. `index.php`, `demos.php` e `keydifferentiators.php` também referenciam `assets/vendor/` (cópia local de Tailwind/Lucide/fontes) que **não está nesta árvore**; o CSS crítico fica inline, então a página inicial continua com boa aparência, mas alguns ícones podem faltar.

---

## 3. Instalação

Substitua `SEU_DOMINIO` e o caminho se usar outro.

```bash
sudo mkdir -p /var/www/democenter
sudo git clone https://github.com/lmatus-nsk/democenter-partner-edition-pt-br.git /var/www/democenter
```

O repositório é privado, então o `git clone` vai pedir credenciais:

- **Username:** seu **usuário do GitHub** (ex.: `joao-silva`), **não** o seu e-mail.
- **Password:** um **Personal Access Token (classic)**, **não** a senha da conta. O GitHub não aceita senha via HTTPS.

**Como gerar o token:**

1. No GitHub, acesse **Settings → Developer settings → Personal access tokens → Tokens (classic)**.
2. Clique em **Generate new token (classic)**.
3. Dê um nome (ex.: `democenter`) e defina a expiração.
4. Marque o escopo **`repo`**.
5. Clique em **Generate token** e copie o valor (começa com `ghp_`). Ele só aparece uma vez.
6. Cole o token no campo `Password` do `git clone`.

> [!WARNING]
> Gere o token na **sua própria conta**. Use o tipo **classic**: tokens *fine-grained* não acessam repositórios pessoais de terceiros nos quais você é colaborador.

> [!IMPORTANT]
> Antes de clonar, aceite o convite de colaborador (enviado por e-mail ou em `https://github.com/lmatus-nsk/democenter-partner-edition-pt-br/invitations`). Sem isso, o Git retorna `Repository not found`.

```bash
sudo chown -R www-data:www-data /var/www/democenter
```

Copie o exemplo de nginx:

```bash
sudo cp /var/www/democenter/deploy/nginx.conf.example /etc/nginx/sites-available/democenter
sudo nano /etc/nginx/sites-available/democenter   # altere server_name e root
sudo ln -s /etc/nginx/sites-available/democenter /etc/nginx/sites-enabled/democenter
sudo nginx -t && sudo systemctl reload nginx
```

Ajuste o socket do PHP-FPM se sua distro não usar `/run/php/php-fpm.sock` (por exemplo `/run/php/php8.3-fpm.sock`).

HTTPS:

```bash
sudo certbot --nginx -d SEU_DOMINIO
```

### Verificar

1. Abra `https://SEU_DOMINIO/` → página inicial em português, **sem** seletor de idioma.
2. Entre em **Quer ver ao vivo?** → `demos.php`.
3. Percorra os cartões (navegação, dados, VPN, IA, rede).
4. `https://SEU_DOMINIO/api/demo-chat.php` deve retornar **404** (Borderbot desligado de propósito).
5. `https://SEU_DOMINIO/api/seats.php` pode retornar 502 se não houver lab Windows; é esperado.

Teste rápido no próprio servidor:

```bash
cd /var/www/democenter
php -S 127.0.0.1:8003
# depois: curl -I http://127.0.0.1:8003/index.php
```

---

## 4. O que há na árvore

```
index.php                 Página inicial (landing)
demos.php                 Portal de demos (hub)
keydifferentiators.php    Quem é a Netskope?
navigationevolution.php
dataprotection.php
vpnevolution.php
aicontrol.php
networktransformation.php
customerreferences.php
reportai.php / reportit.php
oopsyoudidit.php          Página de coaching / bloqueio
api/seats.php             Proxy opcional para o lab Windows
api/demo-chat.php         Desabilitado (404)
assets/                   CSS, JS, JSON, imagens
deploy/nginx.conf.example
```

Não há `.env`, não há banco de dados para criar, não há migrações.

---

## 5. Personalização do serviço

Recomenda-se **editar e adaptar esta edição com ajuda de IA**, para encurtar o tempo de personalização (textos, logos, cartões de demo, ocultar/mostrar seções, i18n, estilos).

Ferramentas recomendadas:

- **[Cursor](https://cursor.com)** — editor com agente sobre o repositório: peça a alteração em linguagem natural, revise o diff e teste no navegador.
- **[Claude](https://claude.ai)** (Anthropic) — para redigir textos, montar o plano de alterações e revisar HTML/PHP/JS antes de aplicá-los.

Fluxo sugerido: abra este repositório no Cursor, descreva a alteração (por exemplo “troque as referências LATAM pelos clientes do partner” ou “oculte o cartão de VPN”), deixe o agente propor o patch, verifique localmente (`php -S` ou o nginx do partner) e só então publique.

Não é necessário um stack de frontend separado: as páginas são PHP/HTML autocontidas. Uma mudança de texto ou de um cartão costuma ser em um único arquivo.

---

## 6. Segurança ao publicar

- Não subir `/etc/demo-chat.env` nem chaves da AWS (esta edição não as usa).
- Não deixar `*.bak`, `.tgz` nem backups dentro do document root.
- O nginx do exemplo já bloqueia `*.bak`, `*.tgz`, `*.env`.
- O site não tem autenticação: trate-o como portal de demo, não como intranet com segredos.

---

## 7. Licença

Código proprietário da Netskope. Uso limitado ao partner autorizado. Não redistribuir nem publicar como open source.
