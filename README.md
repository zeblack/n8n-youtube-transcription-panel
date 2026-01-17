# n8n YouTube Transcription Panel

Painel completo de transcrição e tradução de vídeos do YouTube com conversão texto-para-fala e upload automático para Google Drive.

## 📋 Descrição

Sistema automatizado que transcreve vídeos do YouTube, traduz o conteúdo para múltiplos idiomas, converte para áudio usando Google Text-to-Speech e faz upload dos arquivos para o Google Drive. Ideal para criadores de conteúdo, educadores e empresas que precisam de versões multilíngues de seus vídeos.

## ✨ Funcionalidades

### 🎥 Transcrição Automática
- **API Gratuita**: Usa Supadata.ai (sem rate limits)
- **Processamento Rápido**: Extração eficiente de legendas
- **Limpeza Automática**: Remove timestamps e formatação

### 🌍 Tradução Multilíngue
- **Idiomas Suportados**: Inglês, Espanhol, Português, Francês, Alemão
- **Tradução por Chunks**: Processa textos longos em partes
- **Modelo GPT-3.5**: Traduções precisas e naturais
- **Preservação de Contexto**: Mantém significado original

### 🔊 Conversão Texto-para-Fala
- **Google TTS**: Vozes neurais de alta qualidade
- **Vozes Específicas por Idioma**:
  - Inglês: `en-US-Neural2-F`
  - Espanhol: `es-ES-Neural2-A`
- **Formato MP3**: Compatível com todos os players
- **Nomenclatura Inteligente**: Arquivos com título do vídeo + idioma + timestamp

### ☁️ Upload Automático
- **Google Drive**: Armazenamento em nuvem
- **Links Diretos**: URLs para visualização e download
- **Organização**: Pasta configurável
- **Metadados**: Informações completas dos arquivos

### 🎨 Interface de Formulário
- **Form Trigger**: Interface web amigável
- **Seleção de Idiomas**: Dropdowns para escolha fácil
- **Validação**: Campos obrigatórios
- **Resposta HTML**: Página de sucesso com links

## 🚀 Requisitos

- n8n instalado (self-hosted ou cloud)
- API Key do Supadata (gratuita)
- API Key do OpenRouter (para GPT-3.5)
- API Key do Google Cloud (para TTS)
- Conta Google Drive com OAuth2

## 📦 Instalação

1. **Importar o Workflow**
   ```bash
   # No n8n, vá em Workflows > Import from File
   # Selecione o arquivo workflow.json
   ```

2. **Configurar Credenciais**
   
   Configure as seguintes credenciais no n8n:
   
   - **Google Drive OAuth2**: Para upload de arquivos
   - Nenhuma credencial adicional necessária (APIs usam keys na URL/header)

3. **Obter API Keys**
   
   - **Supadata**: https://supadata.ai/ (gratuito)
   - **OpenRouter**: https://openrouter.ai/keys
   - **Google Cloud TTS**: https://console.cloud.google.com/

4. **Atualizar API Keys no Workflow**
   
   - **Supadata**: Nó "Get YouTube Transcript" (header `x-api-key`)
   - **OpenRouter**: Nós "Translate to English/Spanish" (header `Authorization`)
   - **Google TTS**: Nós "Google TTS English/Spanish" (query parameter `key`)

5. **Configurar Google Drive**
   
   - Atualize `folderId` nos nós "Upload to Drive"
   - Crie uma pasta no Drive e copie o ID da URL

## 🎯 Como Usar

### Via Interface de Formulário

1. **Acesse a URL do Form Trigger**
   - Copie a URL do nó "Form Trigger"
   - Abra no navegador

2. **Preencha o Formulário**
   - **YouTube Video URL**: Cole o link do vídeo
   - **First Language**: Selecione primeiro idioma de tradução
   - **Second Language**: Selecione segundo idioma

3. **Aguarde o Processamento**
   - O workflow processa automaticamente
   - Página de sucesso exibe links dos arquivos

### Via Webhook (Programático)

Faça um POST para a URL do webhook:

```json
{
  "YouTube Video URL": "https://www.youtube.com/watch?v=VIDEO_ID",
  "First Language": "English",
  "Second Language": "Spanish"
}
```

## 📊 Fluxo de Processamento

```
Form Trigger (recebe requisição)
    ↓
Extract Parameters (mapeia idiomas)
    ↓
Get YouTube Transcript (Supadata API)
    ↓
Process & Clean Transcript (limpa e divide em chunks)
    ↓
    ├─→ Translate to English (GPT-3.5)
    │       ↓
    │   Extract English
    │       ↓
    │   Aggregate English
    │       ↓
    │   Merge & Clean English
    │       ↓
    │   Debug & Validate English
    │       ↓
    │   Google TTS English
    │       ↓
    │   Process Audio English
    │       ↓
    │   Upload to Drive - English
    │
    └─→ Translate to Spanish (GPT-3.5)
            ↓
        Extract Spanish
            ↓
        Aggregate Spanish
            ↓
        Merge & Clean Spanish
            ↓
        Debug & Validate Spanish
            ↓
        Google TTS Spanish
            ↓
        Process Audio Spanish
            ↓
        Upload to Drive - Spanish
    ↓
Merge Uploads
    ↓
Consolidate Results
    ↓
Prepare HTML Response
    ↓
Respond to Webhook
```

## ⚙️ Configuração Avançada

### Adicionar Novo Idioma

1. **Atualizar Form Trigger**
   ```javascript
   // Adicionar opção no dropdown
   {
     "option": "French"
   }
   ```

2. **Criar Nós de Tradução**
   - Duplicar "Translate to English"
   - Atualizar prompt do sistema para francês
   - Ajustar código de idioma: `fr`

3. **Criar Nós TTS**
   - Duplicar "Google TTS English"
   - Atualizar `languageCode`: `fr-FR`
   - Selecionar voz neural francesa

4. **Atualizar Extract Parameters**
   ```javascript
   const langMap = {
     // ... idiomas existentes
     'French': 'fr'
   };
   ```

### Ajustar Tamanho dos Chunks

No nó "Process & Clean Transcript":

```javascript
const chunkSize = 3000; // Padrão
// Ajuste conforme necessário
const chunkSize = 5000; // Chunks maiores (menos requisições)
const chunkSize = 2000; // Chunks menores (mais precisão)
```

### Personalizar Vozes TTS

Consulte [Google TTS Voices](https://cloud.google.com/text-to-speech/docs/voices) e atualize:

```json
{
  "voice": {
    "languageCode": "en-US",
    "name": "en-US-Wavenet-D" // Voz masculina
  }
}
```

### Mudar Pasta do Drive

1. Crie pasta no Google Drive
2. Copie ID da URL: `https://drive.google.com/drive/folders/FOLDER_ID`
3. Atualize nos nós "Upload to Drive":

```json
{
  "folderId": {
    "value": "SEU_FOLDER_ID_AQUI"
  }
}
```

## 🔍 Debug e Logs

O workflow inclui logs detalhados:

```javascript
console.log('Estrutura recebida da API:', ...);
console.log('Processando X segmentos de transcrição');
console.log('Texto extraído:', ...);
console.log('Criados X chunks para tradução');
console.log('English translation length:', ...);
```

Monitore execuções no n8n para troubleshooting.

## 🐛 Troubleshooting

### Erro ao obter transcrição
- Verifique se vídeo tem legendas disponíveis
- Confirme API key do Supadata
- Teste URL do vídeo manualmente

### Tradução incompleta
- Verifique tamanho dos chunks (muito grande pode falhar)
- Confirme que todos os chunks foram processados
- Veja logs de agregação

### TTS falha
- Confirme API key do Google Cloud
- Verifique se texto não excede 5000 caracteres
- Teste com texto menor primeiro

### Upload para Drive falha
- Reautentique OAuth2 do Google Drive
- Verifique permissões da pasta
- Confirme que folder ID está correto

### Apenas 1 arquivo ao invés de 2
- Verifique se ambos os idiomas foram selecionados
- Veja logs do "Consolidate Results"
- Confirme que ambos os fluxos completaram

## 🔒 Segurança

⚠️ **IMPORTANTE**:

- **Nunca** exponha API keys no código
- Use credenciais do n8n sempre que possível
- Implemente rate limiting no formulário
- Valide URLs de vídeo antes de processar
- Monitore uso das APIs para evitar custos

## 💰 Custos

### APIs Gratuitas
- **Supadata**: Gratuito (sem rate limits)

### APIs Pagas
- **OpenRouter (GPT-3.5)**: ~$0.0015 por 1K tokens
- **Google TTS**: ~$4 por 1 milhão de caracteres
- **Google Drive**: Gratuito (até 15GB)

**Estimativa por vídeo de 10 minutos:**
- Transcrição: Gratuito
- Tradução (2 idiomas): ~$0.05
- TTS (2 idiomas): ~$0.02
- **Total: ~$0.07 por vídeo**

## 🎓 Casos de Uso

### Criadores de Conteúdo
- Legendas automáticas para vídeos
- Versões multilíngues de podcasts
- Acessibilidade para audiências globais

### Educadores
- Transcrições de aulas
- Material de estudo em múltiplos idiomas
- Audiobooks de conteúdo educacional

### Empresas
- Treinamentos multilíngues
- Documentação de vídeos
- Acessibilidade corporativa

## 🤝 Contribuindo

Contribuições são bem-vindas! Áreas para melhoria:
- Adicionar mais idiomas
- Suporte a outros serviços de TTS
- Interface mais rica
- Processamento em batch

## 📄 Licença

MIT License - veja o arquivo LICENSE para detalhes.

## 🆘 Suporte

- [Documentação n8n](https://docs.n8n.io/)
- [Supadata API Docs](https://supadata.ai/docs)
- [OpenRouter Docs](https://openrouter.ai/docs)
- [Google TTS Docs](https://cloud.google.com/text-to-speech/docs)

---

**Desenvolvido com ❤️ para criadores de conteúdo**
