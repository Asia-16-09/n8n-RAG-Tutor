# n8n RAG Tutor v3

AI-powered language tutor built with n8n, featuring Retrieval-Augmented Generation (RAG), voice message support, and text-to-speech responses.

## Features

- **Multi-language Support**: Works with Romanian, Russian, and English
- **Voice Processing**: Handles both text and voice messages from Telegram
- **RAG Integration**: Uses Supabase vector database for educational content retrieval
- **Text-to-Speech**: Converts AI responses to voice messages
- **Memory System**: Maintains conversation history with PostgreSQL
- **Smart Response**: Only sends voice responses for messages under 800 characters

## Architecture

### Main Components:
- **Telegram Bot**: Handles user interactions
- **OpenAI GPT-4o-mini**: Main language model for tutoring
- **Whisper API**: Speech-to-text transcription
- **OpenAI TTS**: Text-to-speech generation
- **Supabase Vector Store**: Educational materials storage
- **PostgreSQL**: Chat memory storage

### Workflow Flow:
1. User sends text/voice message to Telegram bot
2. Voice messages are transcribed using OpenAI Whisper
3. Message is processed by AI Agent with RAG context
4. Response is generated and sent as text
5. For short responses (≤800 chars), TTS audio is also generated

## Prerequisites

### Required Services:
- [n8n](https://n8n.io/) instance (cloud or self-hosted)
- [OpenAI API](https://openai.com/api/) account
- [Telegram Bot](https://core.telegram.org/bots#creating-a-new-bot) token
- [Supabase](https://supabase.com/) project
- PostgreSQL database (can use Supabase)

### Required Credentials in n8n:
- **Telegram API**: Bot token
- **OpenAI API**: API key with GPT-4, Whisper, and TTS access
- **Supabase API**: Project URL and service role key
- **PostgreSQL**: Connection details for chat memory

## Installation

### Step 1: Create Telegram Bot
1. Message [@BotFather](https://t.me/botfather) on Telegram
2. Create new bot with `/newbot`
3. Save the bot token for n8n configuration

### Step 2: Set up Supabase
1. Create new Supabase project
2. Create table for vector storage:
```sql
-- Enable vector extension
create extension if not exists vector;

-- Create documents table
create table documents (
    id bigserial primary key,
    content text,
    metadata jsonb,
    embedding vector(1536)
);

-- Create index for similarity search
create index on documents using ivfflat (embedding vector_cosine_ops)
with (lists = 100);

-- Create match function
create or replace function match_documents(
    query_embedding vector(1536),
    match_threshold float,
    match_count int
)
returns table (
    id bigint,
    content text,
    metadata jsonb,
    similarity float
)
language sql stable
as $$
    select
        documents.id,
        documents.content,
        documents.metadata,
        1 - (documents.embedding <=> query_embedding) as similarity
    from documents
    where 1 - (documents.embedding <=> query_embedding) > match_threshold
    order by similarity desc
    limit match_count;
$$;
```

### Step 3: Set up PostgreSQL for Memory
1. Use existing Supabase database or separate PostgreSQL instance
2. n8n will automatically create required tables for chat memory

### Step 4: Configure n8n Workflow
1. Import `TutRomTTS-workflow.json` into your n8n instance
2. Configure credentials:
   - **Telegram API**: Add your bot token
   - **OpenAI API**: Add your API key
   - **Supabase**: Add project URL and service role key
   - **PostgreSQL**: Add connection details

### Step 5: Configure Nodes
1. **Telegram Trigger**: 
   - Test webhook connection
   - Enable webhook
2. **Supabase Vector Store**:
   - Set table name to `documents`
   - Configure embedding model (text-embedding-ada-002)
3. **PostgreSQL Memory**:
   - Test connection
   - Set context window length (default: 10)

### Step 6: Populate Knowledge Base
1. Upload educational materials to Google Drive (optional workflow included)
2. Or manually add documents to Supabase vector store
3. Content should be tagged with CEFR levels and topics

## Configuration Options

### AI Agent System Message
The system prompt defines the tutor's behavior. Key aspects:
- Identifies student level and needs
- Retrieves appropriate learning content
- Tracks progress using chat memory
- Provides preliminary advice with teacher disclaimer

### TTS Settings
- Model: `tts-1-hd`
- Voice: `shimmer`
- Only activated for responses ≤800 characters

### Memory Settings
- Session ID: Based on Telegram user ID
- Context window: 10 messages
- Automatic cleanup of old sessions

## Usage

### For Students:
1. Start conversation with your Telegram bot
2. Send text or voice messages in Romanian, Russian, or English
3. Receive personalized tutoring responses
4. Get voice responses for shorter replies

### For Administrators:
1. Monitor workflow execution in n8n
2. Update educational content in Supabase
3. Adjust system prompts for different languages/levels
4. Review chat logs for improvements

## Troubleshooting

### Common Issues:

**Webhook not receiving messages:**
- Check Telegram bot token
- Verify webhook URL in n8n
- Ensure n8n instance is publicly accessible

**Voice transcription failing:**
- Verify OpenAI API key has Whisper access
- Check audio file format support
- Monitor API quotas

**RAG not finding relevant content:**
- Verify Supabase vector store setup
- Check embedding model consistency
- Ensure documents are properly indexed

**TTS not working:**
- Verify OpenAI API key has TTS access
- Check character limit (800 chars)
- Monitor API quotas

### Logs and Debugging:
- Use n8n execution logs to trace workflow steps
- Check individual node outputs for data flow
- Monitor API response codes and error messages

## Customization

### Adding New Languages:
1. Update system message with language instructions
2. Add language-specific educational content
3. Test voice recognition for target language

### Modifying Response Length:
1. Adjust character limit in "Check Text Length" node
2. Consider TTS API limits and costs
3. Update user experience accordingly

### Adding New Tools:
1. Create additional n8n tool nodes
2. Update AI Agent tool connections
3. Modify system message to describe new capabilities

## Contributing

1. Fork the repository
2. Create feature branch
3. Test changes thoroughly
4. Submit pull request with detailed description

## Security Notes

- Never commit actual API keys or credentials
- Use n8n's credential system for sensitive data
- Regularly rotate API keys
- Monitor usage quotas and costs
- Implement rate limiting for production use

## License

MIT License - see LICENSE file for details

## Support

For issues and questions:
1. Check troubleshooting section
2. Review n8n documentation
3. Open GitHub issue with detailed description
4. Include workflow execution logs when possible
