# domain-parser

Script em Bash para extração e análise de domínios a partir de páginas HTML. Útil para reconhecimento passivo, auditoria de segurança e detecção de injeções maliciosas em sites.

## O que faz

- Baixa uma página web via `wget`
- Extrai todos os domínios referenciados nos atributos `href`
- Mostra o contexto HTML de onde cada domínio foi extraído (número da linha e trecho do código)
- Resolve o endereço IP de cada domínio via `dig` ou `host`

## Caso de uso real

Durante um teste com o site de uma prefeitura brasileira, o script identificou dois domínios de cassino/apostas (`casino.unoregler.com` e `rocketpot.io`) injetados no HTML via plugin WordPress comprometido:

```html
<div class="wpseoinj-block" data-wpseoinj-id="1">
    <a href="https://rocketpot.io/" target="_blank" rel="noopener">...</a>
</div>
```

Essa técnica é conhecida como **SEO Poisoning** ou **Black Hat SEO Injection**: links ocultos são inseridos em sites de alta autoridade (como `.gov.br`) para manipular o ranking de busca do Google em favor de sites maliciosos. O bloco é invisível para visitantes humanos, mas rastreado normalmente por bots de busca.

## Dependências

- `wget`
- `dig` (pacote `dnsutils`) — preferencial
- `host` — usado como fallback se `dig` não estiver disponível

## Uso

```bash
chmod +x parser.sh
./parser.sh
```

O script solicita a URL interativamente:

```
=== PARSING DE DOMINIOS ===
Digite o endereco para buscar os dominios-host:
https://exemplo.mg.gov.br
```

## Exemplo de saída

```
========================================
 DOMINIOS, CONTEXTO E IPs
========================================

[DOMINIO] casino.unoregler.com
----------------------------------------
[CONTEXTO]
  linha 1635: <div class="wpseoinj-block" data-wpseoinj-id="1"><p><a href="https://casino.unoregler.com/...
[IP] 104.21.37.93
----------------------------------------

[DOMINIO] cdnjs.cloudflare.com
----------------------------------------
[CONTEXTO]
  linha 12: <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
[IP] 104.17.25.14
----------------------------------------
```

## Observações

- O script não realiza nenhuma requisição além do `wget` inicial — é uma análise passiva do HTML
- Arquivos temporários são criados com `mktemp` e removidos automaticamente ao final via `trap EXIT`
- A saída não usa emojis ou caracteres UTF-8 especiais para garantir compatibilidade com qualquer terminal
