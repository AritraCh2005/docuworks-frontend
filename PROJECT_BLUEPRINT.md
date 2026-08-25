# BSK Assistant - Comprehensive Code Flow Documentation

## 🏛️ System Overview

The BSK (Bangla Sahayata Kendra) Assistant is a sophisticated RAG (Retrieval-Augmented Generation) chatbot system built with Streamlit. It helps operators provide information about government services by leveraging a vector database of documents and AI-powered conversation capabilities.

---

## 📊 System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                           BSK ASSISTANT                         │
├─────────────────────────────────────────────────────────────────┤
│  Frontend (Streamlit UI)                                       │
│  ├── Main App (app.py)                                         │
│  ├── Chat Interface (ui/components/chat_interface.py)          │
│  ├── Sidebar (ui/components/sidebar.py)                        │
│  └── Vector Operations (ui/pages/vector_operations.py)         │
├─────────────────────────────────────────────────────────────────┤
│  Logic Layer                                                   │
│  ├── Chat Orchestration (services/chat_service.py)             │
│  ├── RAG Engine (core/rag_engine.py)                           │
│  ├── Memory Manager (core/memory_manager.py)                   │
│  └── Vector Operations (core/vector_operations.py)             │
├─────────────────────────────────────────────────────────────────┤
│  Models Layer                                                  │
│  ├── Text Embeddings (models/embeddings.py)                    │
│  └── LLM Models (models/llm_models.py)                         │
├─────────────────────────────────────────────────────────────────┤
│  Data Layer                                                    │
│  ├── Vector Store (core/vector_store.py)                       │
│  ├── Chat History (data/chat_history.json)                     │
│  └── Assets (assets/)                                          │
├─────────────────────────────────────────────────────────────────┤
│  Infrastructure                                                │
│  ├── Configuration (config/settings.py)                        │
│  ├── Logging (utils/logger.py)                                │
│  └── Helpers (utils/helpers.py)                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Application Entry Point & Flow

### 1. **Application Startup** (`app.py`)

**Purpose**: Main entry point that initializes the application and routes to appropriate pages.

**Flow**:
1. **Logging Setup**: Initializes comprehensive logging system
2. **Page Configuration**: Sets Streamlit page config (title, icon, layout)
3. **Session State Initialization**: Creates `current_page` state (defaults to "chat")
4. **Page Routing**: Routes to either chatbot page or vector operations page
5. **Error Handling**: Defaults to chat page for unknown routes

**Key Functions**:
- `main()`: Main application orchestrator
- Automatic page routing based on session state

---

## 💬 Chat System Architecture

### 2. **Chat Management System** (`services/chat_service.py`)

**Purpose**: Complete chat lifecycle management with UUID-based chat identification.

#### **Core Features**:

##### **Chat Creation**:
```python
def create_new_chat(self, custom_id: Optional[str] = None) -> str
```
- Generates UUID-based unique chat IDs
- Creates fresh chat metadata structure
- Initializes empty message array
- Sets creation/update timestamps
- Ensures uniqueness across all chats

##### **Chat Storage Structure** (Generalized):
```json
{
  "chat_id_uuid": {
    "created_at": "ISO-timestamp",
    "updated_at": "ISO-timestamp",
    "title": "Chat Title",
    "messages": [
      {
        "seq": 1,
        "role": "user|Virtual Assistant",
        "content": "Message content",
        "timestamp": "ISO-timestamp"
      }
      // ... more messages ...
    ],
    "message_count": N
  }
}
```

##### **Message Management**:
- **Save Messages**: `save_message_to_chat()` - Stores user/AI messages with UUIDs
- **Retrieve Messages**: `get_chat_messages()` - Loads chat conversation history
- **Auto-Title Generation**: Updates chat titles from first user message

##### **Chat Operations**:
- **List Chats**: `get_all_chats()` - Returns sorted chat list (newest first)
- **Delete Chats**: `delete_chat()` - Removes chat and all messages
- **Chat Summaries**: `get_chat_summary()` - Provides metadata overview
- **Cleanup**: `cleanup_empty_chats()` - Removes chats with no messages

---

### 3. **Memory Management System** (`core/memory_manager.py`)

**Purpose**: Isolated conversation memory for each chat with intelligent context management.

#### **Memory Architecture**:

##### **Chat Isolation**:
```python
chat_memories: Dict[str, ConversationBufferWindowMemory] = {}
```
- Each chat gets independent memory instance
- Prevents conversation bleeding between chats
- UUID-based memory identification

##### **Memory Operations**:

**Memory Creation**:
```python
def create_new_chat_memory(self, chat_id: str) -> str
```
- Creates fresh `ConversationBufferWindowMemory` instance
- Configurable window size (default: 6 messages)
- Automatic memory instance management

**Context Switching**:
```python
def switch_to_chat(self, chat_id: str) -> bool
```
- Switches active memory context to specific chat
- Auto-creates memory if doesn't exist
- Updates current chat tracking

**Memory Persistence**:
```python
def save_context(self, input_data: Dict, output_data: Dict) -> bool
```
- Saves conversation pairs to active chat memory
- Maintains conversation continuity
- Error-tolerant with fallback handling

##### **Advanced Memory Features**:
- **Memory Loading**: `load_memory_variables()` - Retrieves conversation history
- **Memory Cleanup**: `clear_current_chat_memory()` - Clears active chat memory
- **Memory Optimization**: Periodic cleanup of inactive memories
- **Health Monitoring**: Tracks memory usage and performance

---

## 🤖 RAG Engine & AI Processing

### 4. **RAG Engine** (`core/rag_engine.py`)

**Purpose**: Core AI processing engine that combines retrieval and generation for contextual responses.

#### **RAG Components**:

##### **Enhanced Prompt Template**:
```python
ChatPromptTemplate.from_messages([
            ("system", SYSTEM_PROMPT),
            ("human", "use this information to answer queries: \n{context}"),
            MessagesPlaceholder(variable_name="history"),
            ("human", "Current question: {input} \n "
             "Language in which reponse should be: {language}"),
        ])
```

##### **Context Retrieval & Enhancement**:
```python
def _get_enhanced_context(self, documents: List, query: str, chat_id: Optional[str] = None)
```
**Process**:
1. **Document Processing**: Extracts content and metadata from retrieved documents
2. **Source Attribution**: Adds source information to each context piece


**Output Structure**:
```python
{
    "context": "formatted_context_with_sources",
    "sources": [{"source": "filename", "page": "page_number"}],
}
```

##### **Query Processing Pipeline**:
```python
def process_query(self, query: str, chat_id: Optional[str] = None)
```

**Processing Steps**:
1. **Query Validation**: Checks query length, content, and format
2. **Memory Context Setup**: Ensures correct chat memory is active
3. **Chain Availability Check**: Verifies RAG chain is functional
4. **Document Retrieval**: Gets relevant documents from vector store
5. **Context Enhancement**: Processes and formats retrieved context
6. **AI Generation**: Streams response using LLM with context
7. **Memory Persistence**: Saves conversation to chat memory
8. **Error Handling**: Comprehensive error recovery and user feedback

**Streaming Response**:
- Real-time response streaming for better UX
- Chunk-based processing with timeout protection
- Full response assembly for memory storage


## 🗄️ Vector Database System

### 5. **Vector Store Management** (`core/vector_store.py`)

**Purpose**: Manages ChromaDB vector database for document storage and retrieval.

#### **Vector Store Architecture**:

##### **Initialization & Loading**:
```python
def _load_vector_store(self)
```
**Process**:
1. **Database Check**: Verifies ChromaDB directory exists
2. **Store Creation**: Creates new store if none exists
3. **Embedding Setup**: Initializes OpenAI embeddings
4. **Retriever Configuration**: Sets up MMR (Maximal Marginal Relevance) retriever

##### **Retriever Configuration**:
```python
search_type="mmr",
search_kwargs={
    "k": 4,           # Number of documents to return
    "fetch_k": 8,     # Number of documents to fetch before MMR
    "lambda_mult": 0.8  # Diversity parameter (updated for better relevance)
}
```


### 6. **Vector Operations** (`core/vector_operations.py`)

**Purpose**: Handles CRUD operations for documents in the vector database.

#### **Document Management**:

##### **Document Addition Pipeline**:
```python
def add_pdf_to_vectorstore(self, uploaded_file, filename: str) -> Dict
```

**Processing Steps**:
1. **Availability Check**: Ensures vector store is operational
2. **Duplicate Prevention**: Checks if document already exists
3. **PDF Processing**: Extracts text using PyPDF2
4. **Text Cleaning**: Removes artifacts and normalizes formatting
   ```python
   def clean_pdf_text(text: str) -> str
   ```
   - Joins broken lines
   - Normalizes whitespace
   - Preserves sentence structure

5. **Text Chunking**: Splits text into manageable chunks
   ```python
   RecursiveCharacterTextSplitter(
       chunk_size=1000,
       chunk_overlap=500,
       length_function=len,
       add_start_index=True
   )
   ```

6. **Metadata Enhancement**: Adds source information to each chunk
7. **Vector Embedding**: Converts text chunks to embeddings
8. **Database Storage**: Persists to ChromaDB with metadata

##### **Document Listing**:
```python
def list_documents(self) -> List[str]
```
- Retrieves all unique document filenames
- Extracts from ChromaDB metadata
- Returns sorted list for UI display

##### **Document Removal**:
```python
def delete_documents_by_filename(self, filename: str) -> Dict
```
- Removes all chunks for specified document
- Maintains database integrity
- Returns operation status

---

## 🎨 User Interface System

### 7. **Main Chat Interface** (`ui/pages/chatbot.py`)

**Purpose**: Primary user interface orchestrating chat functionality.

#### **Session State Management**:
```python
def _initialize_session_state()
```
**Initializes**:
- `current_chat_id`: Active chat tracking
- `current_chat_messages`: Chat message cache
- `pending_new_chat`: New chat creation state
- `memory_loaded_for_chat`: Memory loading tracking
- `chat_history_cache`: Performance optimization cache

#### **Chat Switching Logic**:
```python
def _handle_chat_switching()
```
**Process**:
1. **Memory Check**: Verifies if memory loaded for current chat
2. **Message Loading**: Retrieves chat messages from service
3. **Memory Population**: Loads messages into memory manager
4. **Session Update**: Updates session state with chat messages
5. **Error Handling**: Graceful fallback for loading failures

#### **Performance Optimization**:
**Features**:
- **Health Checks**: Monitors RAG system health every 15 page loads
- **Session Management**: Handles chat switching and memory management
- **Error Handling**: Graceful fallback mechanisms

---

### 8. **Sidebar Navigation** (`ui/components/sidebar.py`)

**Purpose**: Chat history management and navigation interface.

#### **Navigation Features**:
- **Page Switching**: Chat ↔ Vector Operations
- **Visual State Indicators**: Shows current active page
- **Responsive Design**: Adapts to different screen sizes

#### **Chat History Management**:

##### **New Chat Creation**:
```python
def _handle_new_chat()
```
**Process**:
1. **Memory Cleanup**: Clears current chat memory
2. **State Reset**: Resets all chat-related session states
3. **Pending State**: Sets up for new chat creation
4. **UI Refresh**: Triggers interface update

##### **Chat Display**:
```python
def _render_chat_history()
```
**Features**:
- **Chronological Sorting**: Most recent chats first
- **Chat Metadata**: Shows title, message count, timestamps
- **Interactive Selection**: Click to switch between chats
- **Delete Functionality**: Remove unwanted chats

##### **Delete Confirmation**:
```python
def _render_delete_confirmation()
```
- **Safety Mechanism**: Prevents accidental deletions
- **Modal Dialog**: Clean confirmation interface
- **State Management**: Tracks deletion requests

---

### 9. **Chat Interface Components** (`ui/components/chat_interface.py`)

**Purpose**: Core chat interaction components for message display and input.

#### **Message Rendering**:
```python
def render_chat_messages()
```
**Features**:
- **Role-based Styling**: Different styles for user/assistant messages
- **Markdown Support**: Rich text formatting in responses
- **Scrollable Container**: Efficient handling of long conversations
- **Empty State**: Helpful prompts for new chats

#### **Chat Input System**:
```python
def render_chat_input()
```
**Process**:
1. **Input Validation**: Checks message content and length
2. **Chat Creation**: Creates new chat if in pending state
3. **Title Generation**: Updates chat title from first message
4. **Message Saving**: Persists user message to chat history
5. **Memory Management**: Switches to correct chat memory context
6. **AI Processing**: Sends query to RAG engine for processing
7. **Response Streaming**: Displays AI response in real-time
8. **Response Saving**: Persists AI response to chat history

---

### 10. **Vector Operations Interface** (`ui/pages/vector_operations.py`)

**Purpose**: Document management interface for vector database operations.

#### **Document Statistics Display**:
```python
def _display_vector_store_stats()
```
- **Database Health**: Shows vector store operational status
- **Document Count**: Displays total documents in database
- **Status Indicators**: Visual feedback on system state

#### **Document Upload System**:
```python
def _show_add_documents_tab()
```
**Features**:
- **Multi-file Upload**: Support for multiple PDF files
- **File Validation**: Ensures only PDF files accepted
- **Size Display**: Shows file size information
- **Progress Tracking**: Real-time upload progress

#### **Document Processing**:
```python
def _process_document_uploads(uploaded_files)
```
**Process**:
1. **File Validation**: Checks file type and size
2. **Batch Processing**: Handles multiple files sequentially
3. **Progress Display**: Shows processing status for each file
4. **Error Handling**: Manages processing failures gracefully
5. **Success Feedback**: Confirms successful additions
6. **Database Update**: Refreshes statistics after processing

#### **Document Listing**:
```python
def _show_list_documents_tab()
```
- **Document Grid**: Shows all documents in database
- **Metadata Display**: Filename and processing information
- **Delete Functionality**: Remove documents from database
- **Refresh Capability**: Update document list

---

## ⚙️ Configuration & Infrastructure

### 11. **Configuration Management** (`config/settings.py`)

**Purpose**: Centralized configuration for all system components.

#### **Configuration Sections**:

##### **Page Configuration**:
```python
PAGE_CONFIG = {
    "page_title": "BSK Assistant",
    "page_icon": "🏛️",
    "layout": "wide",
    "initial_sidebar_state": "expanded"
}
```

##### **Model Configuration**:
```python
MODEL_CONFIG = {
    "embedding_model": "text-embedding-3-small",
    "chat_model": "gpt-4o-mini-2024-07-18",
    "temperature": 0.3,
    "streaming": True
}
```

##### **Vector Store Configuration**:
```python
VECTOR_STORE_CONFIG = {
    "db_path": "data/chroma_db",
    "search_type": "mmr",
    "k": 4,
    "fetch_k": 8,
    "lambda_mult": 0.5
}
```

##### **Memory Configuration**:
```python
MEMORY_CONFIG = {
    "window_size": 6,
    "return_messages": True,
}
```

---

### 12. **Logging System** (`utils/logger.py`)

**Purpose**: Comprehensive logging for debugging and monitoring.

#### **Logging Features**:
- **Structured Logging**: Consistent log format across components
- **Multiple Levels**: DEBUG, INFO, WARNING, ERROR, CRITICAL
- **File Rotation**: Automatic log file management
- **Component Tracking**: Logger per module for detailed tracking

---

## 🔄 Complete User Flow Examples

### **Scenario 1: Creating a New Chat and Asking a Question**

1. **User Action**: User clicks "New Chat" in sidebar
2. **System Response**: 
   - `_handle_new_chat()` in sidebar.py executes
   - Clears current memory via `memory_manager.clear_current_chat_memory()`
   - Resets session state: `pending_new_chat = True`, `current_chat_id = None`
   - UI refreshes with empty chat interface

3. **User Action**: User types first message "What services are available at BSK?"
4. **System Response**:
   - `render_chat_input()` in chat_interface.py processes input
   - Creates new chat: `chat_service.create_new_chat()` returns UUID
   - Updates chat title: `chat_service.update_chat_title()` with shortened message
   - Saves user message: `chat_service.save_message_to_chat()`
   - Switches memory context: `memory_manager.switch_to_chat()`

5. **AI Processing**:
   - Calls `rag_engine.process_query()` with user message and chat_id
   - Validates query with `_validate_query()`
   - Retrieves relevant documents from vector store
   - Enhances context with `_get_enhanced_context()`
   - Processes through LLM chain with conversation history
   - Streams response back to UI in real-time

6. **Response Handling**:
   - Displays streaming response in chat interface
   - Saves AI response: `chat_service.save_message_to_chat()`
   - Updates memory: `memory_manager.save_context()`
   - Updates chat metadata (timestamp, message count)

### **Scenario 2: Switching Between Existing Chats**

1. **User Action**: User clicks on existing chat in sidebar
2. **System Response**:
   - Sets `current_chat_id` in session state
   - `_handle_chat_switching()` in chatbot.py executes
   - Loads chat messages: `chat_service.get_chat_messages()`
   - Loads messages into memory: `memory_manager.load_chat_messages_from_history()`
   - Updates UI with chat messages

3. **Memory Context**:
   - Switches active memory to selected chat
   - Preserves conversation context for that specific chat
   - Ensures responses consider chat-specific history

### **Scenario 3: Adding Documents to Vector Database**

1. **User Action**: User navigates to Vector Operations page
2. **System Response**:
   - `show_vector_operations_page()` loads
   - Displays current database statistics
   - Shows document upload interface

3. **User Action**: User uploads PDF files
4. **System Response**:
   - `_process_document_uploads()` handles file processing
   - For each file:
     - Extracts text with PyPDF2
     - Cleans text with `clean_pdf_text()`
     - Splits into chunks with RecursiveCharacterTextSplitter
     - Generates embeddings with OpenAI
     - Stores in ChromaDB with metadata
   - Updates database statistics
   - Provides success/error feedback

---

## 🛡️ Error Handling & Recovery

### **Robust Error Handling Mechanisms**:

1. **Memory Management Errors**:
   - Graceful fallback when memory loading fails
   - Automatic memory cleanup for corrupted instances
   - Context preservation across failures

2. **AI Processing Errors**:
   - Rate limit handling with user guidance
   - Timeout management for long-running queries
   - Context processing error recovery

3. **File Processing Errors**:
   - PDF extraction error handling
   - Unsupported file type messages
   - Partial processing recovery

---

## 📊 Performance Optimizations

### **System Performance Features**:

1. **Streaming Responses**:
   - Real-time AI response streaming
   - Chunk-based processing
   - Timeout protection

2. **Database Efficiency**:
   - MMR (Maximal Marginal Relevance) for diverse results
   - Optimized chunk sizes for retrieval
   - Metadata-based filtering

---

## 🔧 Key Technical Decisions

### **Architecture Choices**:

1. **UUID-based Chat IDs**: Ensures uniqueness and prevents ID collisions
2. **Isolated Memory Per Chat**: Prevents conversation bleeding
3. **Streaming AI Responses**: Improves user experience
4. **ChromaDB for Vector Storage**: Open-source, efficient similarity search
5. **Streamlit for UI**: Rapid prototyping and deployment
6. **Comprehensive Logging**: Debugging and monitoring capabilities

### **Security & Privacy**:

1. **API Key Management**: Environment variable-based configuration
2. **Data Isolation**: Chat conversations stored separately
3. **Error Masking**: User-friendly error messages without system details

---

## 📈 System Scalability

### **Scalability Considerations**:

1. **Memory Management**: Automatic cleanup prevents memory leaks
2. **Database Growth**: ChromaDB handles large document collections
3. **Concurrent Users**: Session-based state management
4. **Performance Monitoring**: Health checks and optimization

### **Future Enhancement Areas**:

1. **User Authentication**: Multi-user support with user-specific chats
2. **Advanced Analytics**: Usage tracking and performance metrics
3. **Enhanced Search**: Hybrid search combining semantic and keyword
4. **API Integration**: RESTful API for external system integration

---

## 🎯 Summary

The BSK Assistant is a sophisticated RAG chatbot system designed for government service assistance. It features:

- **Complete Chat Lifecycle Management**: Creation, switching, deletion with UUID-based identification
- **Intelligent Memory Management**: Isolated conversation contexts with automatic optimization
- **Advanced RAG Processing**: Document retrieval, context enhancement, and AI generation
- **Robust Vector Database**: Document storage, processing, and retrieval with ChromaDB
- **User-Friendly Interface**: Streamlit-based UI with real-time interactions
- **Comprehensive Error Handling**: Graceful failures and automatic recovery
- **Performance Optimization**: Caching, streaming, and resource management

The system is designed for reliability, scalability, and ease of use, making it suitable for production deployment in government service environments.
