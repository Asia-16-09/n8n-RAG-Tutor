# n8n RAG Tutor v3

![n8n](https://img.shields.io/badge/n8n-workflow-FF6D00?style=flat&logo=n8n&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4-412991?style=flat&logo=openai&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-Vector--DB-3ECF8E?style=flat&logo=supabase&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-Bot-2AABEE?style=flat&logo=telegram&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Memory-4169E1?style=flat&logo=postgresql&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)

AI-powered language tutor collection built with n8n, featuring Retrieval-Augmented Generation (RAG), voice message support, text-to-speech responses, and advanced learning tools.

## Available Workflows

### TutRomTTS (Romanian Tutor)
- **Focus**: Romanian language learning for Russian/English speakers
- **Features**: Voice transcription, TTS responses, RAG knowledge base
- **Voice**: Generates audio responses for messages under 800 characters
- **Memory**: PostgreSQL chat history with user progress tracking

### English Tutor with RAG
- **Focus**: English language learning (A1-C1 levels)
- **Features**: Calendar booking, internet search, comprehensive learning tools
- **Tools**: Tavily search, Zep memory, calendar integration
- **Advanced**: CEFR level assessment and personalized lesson plans

## Core Features

- **Multi-language Support**: Romanian and English tutoring workflows
- **Voice Processing**: Handles both text and voice messages from Telegram
- **RAG Integration**: Uses Supabase vector database for educational content retrieval
- **Text-to-Speech**: Converts AI responses to voice messages (Romanian workflow)
- **Memory Systems**: PostgreSQL and Zep for conversation history
- **Internet Search**: Real-time content retrieval via Tavily API
- **Calendar Integration**: Lesson booking and scheduling system

## Architecture

### Main Components:
- **Telegram Bot**: Handles user interactions across both workflows
- **OpenAI GPT-4o-mini**: Primary language model for tutoring
- **Whisper API**: Speech-to-text transcription for voice messages
- **OpenAI TTS**: Text-to-speech generation (Romanian workflow)
- **Supabase Vector Store**: Educational materials storage and retrieval
- **PostgreSQL/Zep**: Chat memory and user progress tracking
- **Tavily API**: Real-time internet search for current content

### Workflow Architecture:
1. User sends text/voice message to Telegram bot
2. Voice messages are transcribed using OpenAI Whisper
3. Message is processed by AI Agent with RAG context retrieval
4. Response generation using GPT-4o-mini with personalized context
5. Text response sent immediately to user
6. Optional TTS audio generation for short responses (Romanian workflow)
7. User progress and conversation stored in memory system

## Prerequisites

### Required Services:
- [n8n](https://n8n.io/) instance (cloud or self-hosted)
- [OpenAI API](https://openai.com/api/) account with GPT-4, Whisper, and TTS access
- [Telegram Bot](https://core.telegram.org/bots#creating-a-new-bot) token
- [Supabase](https://supabase.com/) project with vector extension
- PostgreSQL database (can use Supabase or separate instance)
- [Tavily API](https://tavily.com/) key for internet search (English workflow)

### Required n8n Credentials:
- **Telegram API**: Bot token for both workflows
- **OpenAI API**: API key with all required model access
- **Supabase API**: Project URL and service role key
- **PostgreSQL**: Connection details for chat memory
- **Tavily API**: Search API key (English workflow only)

## Quick Start

### 1. Create Telegram Bot
```bash
# Message @BotFather on Telegram
/newbot
# Follow prompts and save the bot token
```

### 2. Set up Supabase Vector Database
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

-- Create similarity search index
create index on documents using ivfflat (embedding vector_cosine_ops)
with (lists = 100);

-- Create match function for RAG queries
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

### 3. Import and Configure Workflows
1. Import desired JSON workflow file into n8n
2. Configure all credentials using n8n's credential system
3. Test individual nodes before activating workflow
4. Populate vector database with educational content

### 4. Workflow-Specific Setup

#### Romanian Tutor (TutRomTTS):
- Configure TTS settings (voice: shimmer, model: tts-1-hd)
- Set character limit for voice responses (default: 800)
- Upload Romanian learning materials to vector database

#### English Tutor:
- Set up Tavily API for internet search functionality
- Configure calendar workflow for lesson booking
- Set up Zep memory service
- Upload CEFR-leveled content to vector database

## Configuration Options

### System Prompts
Both workflows use comprehensive system prompts that define:
- Target language and student levels
- Available tools and their purposes
- Teaching methodology and interaction patterns
- Progress tracking and personalization approaches

### Memory Settings
- **Session Management**: Based on Telegram user ID
- **Context Window**: Configurable message history length
- **Progress Tracking**: Automated error pattern detection
- **Personalization**: Adaptive content based on learning history

### TTS Configuration (Romanian Workflow)
```json
{
  "model": "tts-1-hd",
  "voice": "shimmer",
  "character_limit": 800,
  "language": "auto-detect"
}
```

## Usage Patterns

### For Students:
1. Start conversation with Telegram bot
2. Send messages in target language or native language
3. Request specific help (grammar, vocabulary, pronunciation)
4. Practice with interactive exercises
5. Book lessons through calendar integration (English workflow)

### For Administrators:
1. Monitor workflow execution in n8n dashboard
2. Update educational content in Supabase vector store
3. Analyze usage patterns through memory system
4. Adjust system prompts for different learning objectives

## Customization

### Adding New Languages:
1. Modify system prompts with language-specific instructions
2. Upload appropriate educational content to vector database
3. Test voice recognition accuracy for target language
4. Adjust cultural context in AI responses

### Extending Functionality:
1. Add new tool nodes in n8n workflow
2. Update AI Agent connections
3. Modify system prompts to describe new capabilities
4. Test integration with existing memory system

## Troubleshooting

### Common Issues:

**Telegram webhook not receiving messages:**
- Verify bot token configuration in n8n credentials
- Check webhook URL accessibility
- Ensure n8n instance has public internet access
- Test webhook connection manually

**Voice transcription errors:**
- Verify OpenAI API key has Whisper access
- Check supported audio formats and file size limits
- Monitor API quota usage and billing status
- Test with different audio quality levels

**RAG system not finding relevant content:**
- Verify Supabase vector store configuration
- Check embedding model consistency (text-embedding-ada-002)
- Ensure documents are properly chunked and indexed
- Test similarity search thresholds

**Memory system not persisting conversations:**
- Check PostgreSQL/Zep connection settings
- Verify session ID generation logic
- Monitor database storage and permissions
- Test memory retrieval with sample queries

**TTS generation failing (Romanian workflow):**
- Verify OpenAI API key has TTS model access
- Check character limits and text preprocessing
- Monitor API quota and rate limits
- Test with different voice models

### Performance Optimization:
- Implement caching for frequently accessed content
- Optimize vector search parameters for your content
- Configure appropriate context window sizes
- Monitor and adjust API rate limits

## API Costs Estimation

### Per 1000 Students/Month:
- **GPT-4o-mini**: ~$15-25 (varies by conversation length)
- **Whisper API**: ~$5-10 (voice message transcription)
- **TTS API**: ~$10-15 (Romanian workflow voice responses)
- **Embeddings**: ~$2-5 (content indexing and search)
- **Tavily Search**: ~$3-8 (English workflow internet queries)

Total estimated monthly cost: $35-63 per 1000 active users

## Security Considerations

### Data Protection:
- All credentials stored securely in n8n credential system
- User conversations encrypted in PostgreSQL database
- API keys rotated regularly
- Webhook endpoints secured with HTTPS

### Privacy Compliance:
- User data retention policies configurable
- GDPR-compliant data deletion procedures
- Conversation logging can be disabled
- Personal information handling documented

## Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature/new-language-support`
3. Test changes thoroughly with sample conversations
4. Submit pull request with detailed description
5. Include updated documentation for new features

## License

MIT License - see LICENSE file for complete terms

## Support and Community

### Documentation:
- [n8n Documentation](https://docs.n8n.io/)
- [OpenAI API Guide](https://platform.openai.com/docs)
- [Supabase Vector Guide](https://supabase.com/docs/guides/database/extensions/pgvector)

### Getting Help:
1. Check troubleshooting section above
2. Review n8n community forums
3. Open GitHub issue with:
   - Workflow execution logs
   - Error messages and screenshots
   - Steps to reproduce the issue
   - n8n version and environment details

## Changelog

### v3.0.0
- Added Romanian tutor with TTS functionality
- Enhanced English tutor with calendar integration
- Implemented Tavily search for real-time content
- Added comprehensive documentation and setup guides
- Cleaned all sensitive data for public release
