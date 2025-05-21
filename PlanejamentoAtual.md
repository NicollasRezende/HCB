# MIGRAÇÃO DE NOTÍCIAS HCB PARA LIFERAY

## DADOS DO AMBIENTE DE HOMOLOGAÇÃO

**URL do Liferay:** https://hcb-portal-teste.seatecnologia.com.br

**Credenciais:**
- Usuário: test@liferay.com
- Senha: Se@20hcb24#

**IDs Importantes:**
- ID do Site: 20117
- ID da Estrutura de Notícias: 32533
- ID da Pasta de Notícias: 32559

## URL DO PORTAL HCB E ESTRUTURA

**URL Base:** https://www.hcb.org.br/geral?pagina=1&tam=10

**Detalhes de Paginação:**
- Páginas disponíveis: 94 páginas
- Notícias por página: 10 itens
- Total de notícias: aproximadamente 940

## CAMPOS DA ESTRUTURA DE NOTÍCIAS

**Campos Identificados:**
- `img_destaque`: Imagem principal
- `legenda_destaque`: Legenda da imagem
- `data`: Data de publicação
- `corpo_noticia`: Conteúdo principal

## ARQUIVOS DE APOIO

**Lista de Links:** hcb_links.txt

## CONFIGURAÇÕES PARA O ARQUIVO .ENV

```
LIFERAY_URL=https://hcb-portal-teste.seatecnologia.com.br
LIFERAY_USERNAME=test@liferay.com
LIFERAY_PASSWORD=Se@20hcb24#
LIFERAY_SITE_ID=20117
# Liferay - Structures
LIFERAY_CONTENT_STRUCTURE_ID=30301
LIFERAY_NEWS_STRUCTURE_ID=32533
# Google Sheets
GOOGLE_SHEETS_CREDENTIALS=client_secret.json
SPREADSHEET_ID=1EV8BraH23ePjKFUI-T96UBgegn9-xxuKPvSYxXEotEY
LIFERAY_SECRETARIAT_NAME=HCB
# Update
LIFERAY_FOLDER_ID_NEWS=32559
```

## SCRIPTS A ADAPTAR

1. Adaptação do `field_mapper.py` para os campos específicos do HCB
2. Ajuste do `get_structure_fields.py` para conectar ao ambiente de homologação
3. Adaptação do script principal de migração para o formato das notícias do HCB
