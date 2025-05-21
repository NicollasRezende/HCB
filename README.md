# Relatório Técnico: Mapeamento e Extração de Notícias do HCB para o Liferay

## 1. Visão Geral da Estrutura

O sistema de gestão de conteúdo do Hospital da Criança de Brasília (HCB) utiliza a plataforma Liferay com uma estrutura específica para notícias. Nossa análise identificou os campos essenciais, seus tipos de dados e requisitos para uma migração eficiente do conteúdo web existente.

## 2. Mapeamento de Campos Detalhado

### 2.1 Campos Principais

| Campo | ID Técnico | Tipo | Mapeamento no HTML Atual | Obrigatório |
|-------|------------|------|--------------------------|-------------|
| Título da Notícia | `titulo` (implícito) | Texto | `.noticias.internas .titulo h2` | Sim* |
| Data de Publicação | `data` (implícito) | Data | `.noticias.internas .titulo h5` | Sim* |
| Imagem Destaque | `img_destaque` | Imagem | `.boxConteudo img` (src, alt, title) | Não |
| Legenda da Imagem | `legenda_destaque` | Texto | `.boxConteudo img` (alt/title) | Não |
| Conteúdo | `corpo_noticia` | Rich Text | `.noticias.internas .txt` (HTML completo) | Sim* |

\* *Campos essenciais para o funcionamento adequado do sistema, mesmo que não tecnicamente obrigatórios.*

### 2.2 Campos da Galeria (Carousel)

| Campo | ID Técnico | Tipo | Repetível | Mapeamento no HTML Atual |
|-------|------------|------|-----------|--------------------------|
| Item do Carousel | `galeria` | Fieldset | Sim | Container para cada imagem+legenda |
| → Imagem | `galeria_fotos` | Imagem | Não | Imagens do corpo: `.noticias.internas .txt img` |
| → Legenda | `galeria_legendas` | Texto | Não | Atributos alt/title das imagens ou texto próximo |

## 3. Análise do HTML Atual

### 3.1 Estrutura de Página de Notícia

```
<section class="noticias internas">
  <div class="container">
    <div class="row">
      <div class="col-lg-9 col-md-9 col-sm-9 col-xs-12">
        <article>
          <div class="titulo">
            <h2>[TÍTULO DA NOTÍCIA]</h2>
            <h5>[DATA DA PUBLICAÇÃO]</h5>
            ...
          </div>
          <div class="txt">
            [CONTEÚDO HTML DA NOTÍCIA]
          </div>
        </article>
      </div>
    </div>
  </div>
</section>
```

### 3.2 Padrões Observados

1. **Título e Data**: Consistentemente encontrados nos elementos `.titulo h2` e `.titulo h5`.

2. **Conteúdo**: Sempre dentro do elemento `.txt`, contendo parágrafos e imagens inline.

3. **Imagens no Conteúdo**: 
   - Inseridas diretamente nos parágrafos (`<p><img src="..." titulo=""></p>`)
   - Geralmente sem legenda explícita no HTML, apenas atributos de imagem
   - Formato: `/arquivos/img_[NÚMERO].jpg`

4. **Imagem Destaque**: 
   - Localizada no sidebar direito (`.boxConteudo img`)
   - Reutilizada frequentemente da primeira imagem do conteúdo

## 4. Estratégia de Extração e Transformação

### 4.1 Extração de Dados

1. **Dados Básicos**:
   ```python
   title = soup.select_one(".noticias.internas .titulo h2").text.strip()
   date = soup.select_one(".noticias.internas .titulo h5").text.strip()
   content_html = str(soup.select_one(".noticias.internas .txt"))
   ```

2. **Imagem Destaque**:
   ```python
   featured_img = soup.select_one(".boxConteudo img")
   img_destaque = {
       "url": urljoin(BASE_URL, featured_img.get("src", "")),
       "alt": featured_img.get("alt", ""),
       "title": featured_img.get("title", "")
   }
   ```

3. **Detecção de Imagens para Galeria**:
   ```python
   content_images = soup.select(".noticias.internas .txt img")
   gallery_items = []
   
   for img in content_images:
       gallery_items.append({
           "image": {
               "url": urljoin(BASE_URL, img.get("src", "")),
               "alt": img.get("alt", ""),
               "title": img.get("title", "")
           },
           "caption": img.get("alt", "") or img.get("title", "")
       })
   ```

### 4.2 Transformação para o Formato Liferay

```json
{
  "title": "Título da Notícia",
  "datePublished": "15/05/2025",
  "img_destaque": {
    "url": "https://www.hcb.org.br/arquivos/img_1747344842.jpg",
    "alt": "Descrição da imagem",
    "descriptiveURL": "URL descritiva"
  },
  "legenda_destaque": "Legenda da imagem principal",
  "corpo_noticia": "<div class=\"txt\">...</div>",
  "galeria": [
    {
      "galeria_fotos": {
        "url": "https://www.hcb.org.br/arquivos/img_1747344982.jpg",
        "alt": "Descrição da imagem 1",
        "descriptiveURL": "URL descritiva 1"
      },
      "galeria_legendas": "Legenda da imagem 1"
    },
    {
      "galeria_fotos": {
        "url": "https://www.hcb.org.br/arquivos/img_1747345067.jpg",
        "alt": "Descrição da imagem 2",
        "descriptiveURL": "URL descritiva 2"
      },
      "galeria_legendas": "Legenda da imagem 2"
    }
  ]
}
```

## 5. Desafios e Soluções

| Desafio | Solução Proposta |
|---------|------------------|
| Imagens sem legendas explícitas | Utilizar atributos alt/title ou texto contextual próximo à imagem |
| Imagens incorporadas no conteúdo HTML | Identificar e extrair para o carousel, mantendo referência no conteúdo |
| Manipulação do Rich Text | Preservar integridade do HTML, garantindo que links e formatação sejam mantidos |
| Ordem das imagens no carousel | Manter a sequência original do documento para preservar a narrativa visual |
| Diferentes formatos de data | Padronizar no formato esperado pelo Liferay (DD/MM/AAAA) |

## 6. Plano de Implementação

### 6.1 Modificações no Script de Scraping

1. **Aprimorar Extração de HTML**:
   ```python
   def extract_news_detail(soup, url):
       # Código existente...
       
       # Extrair HTML completo da div.txt
       content_element = soup.select_one(".noticias.internas .txt")
       content_html = str(content_element) if content_element else ""
       
       # Modificações para processamento de galeria
       gallery_items = []
       if content_element:
           images = content_element.select("img")
           for img in images:
               gallery_items.append({
                   "image": urljoin(BASE_URL, img.get("src", "")),
                   "caption": img.get("alt", "") or img.get("title", "")
               })
       
       # Retornar com campos adicionais
       return {
           # Campos existentes...
           'content_html': content_html,
           'gallery_items': gallery_items
       }
   ```

2. **Criar Processador para Formato Liferay**:
   ```python
   def format_for_liferay(news_data):
       """Converte dados extraídos para o formato JSON do Liferay"""
       return {
           # Mapeamento conforme estrutura do Liferay
           "availableLanguageIds": ["pt_BR"],
           "defaultLanguageId": "pt_BR",
           # Outros campos...
           "img_destaque": {/* Dados da imagem */},
           "legenda_destaque": news_data.get("featured_image_alt", ""),
           "corpo_noticia": news_data.get("content_html", ""),
           "galeria": [
               {
                   "galeria_fotos": {/* Dados da imagem */},
                   "galeria_legendas": item["caption"]
               } for item in news_data.get("gallery_items", [])
           ]
       }
   ```

### 6.2 Etapas de Desenvolvimento

1. **Fase 1**: Adaptar scraper para extrair HTML e metadados conforme mapeamento
2. **Fase 2**: Desenvolver processador para converter dados para formato Liferay
3. **Fase 3**: Implementar detecção e processamento de imagens para galeria
4. **Fase 4**: Testes de validação com amostra de conteúdo
5. **Fase 5**: Processamento em lote de todo o conteúdo
6. **Fase 6**: Importação para o Liferay via API

## 7. Recomendações Adicionais

1. **Otimização de Imagens**: Considerar processamento para padronizar tamanhos e formatos
2. **Metadados SEO**: Extrair e mapear atributos alt/title para melhorar acessibilidade
3. **Validação de HTML**: Garantir que o HTML extraído seja válido e compatível com o editor Liferay
4. **Controle de Duplicação**: Implementar detecção de conteúdos duplicados ou similares
5. **Prevenção de Perda de Dados**: Manter backups dos conteúdos originais antes da migração

Este mapeamento técnico permite a transferência eficiente dos dados do site atual do HCB para o sistema Liferay, preservando a integridade do conteúdo e otimizando o processo de migração.
