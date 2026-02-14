# Implementation Plan: Fabino Medical Decoder

## Overview

This implementation plan breaks down the Fabino Medical Decoder into discrete coding tasks that build incrementally. The system is a multi-agent AI pipeline that transforms medical documents into accessible, multilingual audio-enabled summaries. The implementation follows a bottom-up approach: core data models → individual agents → orchestration → API layer → frontend integration.

## Tasks

- [ ] 1. Set up project structure and dependencies
  - Create Python project structure with src/, tests/, and config/ directories
  - Set up virtual environment and requirements.txt with core dependencies (FastAPI, Pydantic, boto3, openai, googletrans, gTTS, hypothesis, pytest)
  - Configure environment variables for API keys (AWS, OpenAI, Google Translate)
  - Set up logging configuration
  - _Requirements: All (foundational)_

- [ ] 2. Implement core data models
  - [ ] 2.1 Create document and upload models
    - Implement DocumentType, Language, UploadFile, UploadResult, ValidationResult models using Pydantic
    - Add field validation and constraints (file size limits, enum values)
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_
  
  - [ ]* 2.2 Write property test for document models
    - **Property 1: Supported format acceptance**
    - **Property 2: Unsupported format rejection**
    - **Property 4: File size boundary validation**
    - **Validates: Requirements 1.1, 1.2, 1.3, 1.4, 1.5**
  
  - [ ] 2.3 Create extraction and medical context models
    - Implement OCRResult, ExtractionResult, ContextType, MedicalEntity, ContextChunk, MedicalContext models
    - Add confidence score validation and entity position tracking
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6_
  
  - [ ] 2.4 Create summary, translation, and audio models
    - Implement SummarySection, PlainLanguageSummary, StructuredSummary, TranslatedSummary, AudioFile, AudioMetadata models
    - Add reading level validation and audio size constraints
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.6, 4.2, 5.1, 5.6_
  
  - [ ] 2.5 Create pipeline orchestration models
    - Implement PipelineStage, PipelineState, ProcessingResult, UserPreferences, DocumentMetadata models
    - Add progress tracking and error state handling
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5, 6.6_

- [ ] 3. Implement Document Upload Service
  - [ ] 3.1 Create DocumentUploadService class with file validation
    - Implement upload_document() method with MIME type checking
    - Implement validate_file() method for format and size validation
    - Add supported format constants (PDF, JPG, PNG, TXT)
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_
  
  - [ ]* 3.2 Write unit tests for upload validation
    - Test valid format acceptance for each supported type
    - Test invalid format rejection with error messages
    - Test file size boundary conditions (exactly 10MB, over 10MB)
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

- [ ] 4. Implement Extraction Agent
  - [ ] 4.1 Create ExtractionAgent class with PDF extraction
    - Implement extract_text() method with document type routing
    - Implement extract_from_pdf() using pdfplumber
    - Handle multi-page PDF documents
    - _Requirements: 1.1, 1.6, 2.2_
  
  - [ ] 4.2 Add OCR extraction for images
    - Implement extract_from_image() using Tesseract OCR
    - Implement preprocess_image() for image enhancement (grayscale, contrast, noise reduction)
    - Add confidence score calculation
    - _Requirements: 1.2, 2.2, 8.2_
  
  - [ ]* 4.3 Write property test for extraction agent
    - **Property 3: Multi-page processing completeness**
    - **Property 6: Critical information preservation**
    - **Validates: Requirements 1.6, 2.2**
  
  - [ ]* 4.4 Write unit tests for OCR edge cases
    - Test low-confidence OCR warning generation
    - Test image preprocessing effectiveness
    - Test text extraction from various image qualities
    - _Requirements: 8.2_

- [ ] 5. Implement Storage Service with encryption
  - [ ] 5.1 Create StorageService class with encryption
    - Implement store_document() with AES-256 encryption
    - Implement retrieve_document() with decryption
    - Implement schedule_deletion() for automatic 24-hour deletion
    - Implement delete_all_user_data() for user-initiated deletion
    - _Requirements: 7.2, 7.3, 7.5_
  
  - [ ]* 5.2 Write property tests for storage service
    - **Property 28: Transport encryption** (verify TLS configuration)
    - **Property 29: Storage encryption** (verify AES-256)
    - **Property 30: Automatic deletion scheduling**
    - **Property 31: User-initiated deletion**
    - **Validates: Requirements 7.1, 7.2, 7.3, 7.5**

- [ ] 6. Checkpoint - Ensure foundational components work
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 7. Implement RAG Agent with AWS Bedrock integration
  - [ ] 7.1 Create RAGAgent class with entity extraction
    - Implement extract_medical_entities() to identify medical terms, drugs, and lab tests
    - Use regex patterns and medical terminology dictionaries for entity recognition
    - _Requirements: 2.1, 2.3, 2.4, 2.5_
  
  - [ ] 7.2 Implement AWS Bedrock Knowledge Base integration
    - Implement query_knowledge_base() for semantic search
    - Implement retrieve_medical_context() with context type routing
    - Configure knowledge base partitions for terminology, drugs, and lab tests
    - Add context aggregation and deduplication logic
    - _Requirements: 2.1, 2.3, 2.4, 2.5, 9.3_
  
  - [ ] 7.3 Add unknown term handling
    - Implement logic to flag terms with insufficient context (empty or low-relevance results)
    - _Requirements: 2.6_
  
  - [ ]* 7.4 Write property tests for RAG agent
    - **Property 5: RAG context retrieval**
    - **Property 7: Entity-specific context retrieval**
    - **Property 8: Unknown term flagging**
    - **Validates: Requirements 2.1, 2.3, 2.4, 2.5, 2.6**

- [ ] 8. Implement Summarization Agent with OpenAI integration
  - [ ] 8.1 Create SummarizationAgent class with prompt engineering
    - Implement generate_summary() with OpenAI GPT-4 API integration
    - Create prompt template that incorporates medical context and simplification instructions
    - Add reading level target (6th grade) to prompt
    - _Requirements: 3.1, 3.2, 3.5, 3.6, 9.4_
  
  - [ ] 8.2 Add summary structuring and organization
    - Implement structure_summary() to parse and organize into sections
    - Implement section detection for diagnosis, medications, test results, instructions, warnings
    - Add critical information emphasis markers
    - _Requirements: 3.3, 3.4_
  
  - [ ] 8.3 Add reading level validation
    - Implement simplify_language() with Flesch-Kincaid grade level calculation
    - Add iterative simplification if reading level exceeds target
    - _Requirements: 3.6_
  
  - [ ]* 8.4 Write property tests for summarization agent
    - **Property 9: Summary generation from context**
    - **Property 10: Medical jargon simplification**
    - **Property 11: Summary section organization**
    - **Property 12: Critical information emphasis**
    - **Property 13: Reading level compliance**
    - **Validates: Requirements 3.1, 3.2, 3.3, 3.4, 3.6**

- [ ] 9. Implement Translation Agent
  - [ ] 9.1 Create TranslationAgent class with Google Translate integration
    - Implement translate_summary() with language preference handling
    - Implement translate_with_retry() with automatic retry logic
    - Add support for Hindi, Tamil, and English
    - _Requirements: 4.1, 4.2, 4.3, 4.5_
  
  - [ ] 9.2 Add translation validation and fallback
    - Implement validate_translation() for quality checks
    - Add fallback to English summary on translation failure
    - _Requirements: 4.4, 4.5, 8.4_
  
  - [ ]* 9.3 Write property tests for translation agent
    - **Property 14: Language preference persistence**
    - **Property 15: Translation execution**
    - **Property 16: Supported language validation**
    - **Property 17: Translation retry and fallback**
    - **Validates: Requirements 4.1, 4.2, 4.3, 4.5**

- [ ] 10. Implement Audio Generation Agent
  - [ ] 10.1 Create AudioGenerationAgent class with gTTS integration
    - Implement generate_audio() with gTTS for text-to-speech conversion
    - Add language-specific voice selection
    - _Requirements: 5.1, 5.2, 9.5_
  
  - [ ] 10.2 Add audio optimization and chunking
    - Implement optimize_audio() to compress audio to meet 5MB size limit
    - Implement chunk_long_text() for summaries over 500 words
    - Set audio format to MP3 with 64 kbps bitrate
    - _Requirements: 5.6_
  
  - [ ]* 10.3 Write property tests for audio generation agent
    - **Property 18: Audio generation from translation**
    - **Property 21: Audio format and size constraints**
    - **Validates: Requirements 5.1, 5.6**

- [ ] 11. Checkpoint - Ensure all agents work independently
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 12. Implement Agent Orchestrator
  - [ ] 12.1 Create AgentOrchestrator class with pipeline management
    - Implement process_document() as main entry point
    - Implement execute_pipeline() with sequential agent execution
    - Define pipeline stage constants and ordering
    - _Requirements: 6.1, 6.3_
  
  - [ ] 12.2 Add progress tracking and state management
    - Implement track_progress() to update pipeline state
    - Emit progress updates at each stage transition
    - Store pipeline state in database or cache
    - _Requirements: 6.4, 9.6_
  
  - [ ] 12.3 Add error handling and retry logic
    - Implement handle_agent_failure() with recovery actions
    - Add 60-second timeout per agent with one retry
    - Implement graceful degradation (partial results on failure)
    - Add detailed error logging
    - _Requirements: 6.2, 6.5, 8.3, 8.5_
  
  - [ ] 12.4 Add output persistence
    - Store all outputs (summary, translation, audio) after pipeline completion
    - Associate outputs with document ID for retrieval
    - _Requirements: 6.6_
  
  - [ ]* 12.5 Write property tests for orchestrator
    - **Property 22: Pipeline stage ordering**
    - **Property 23: Error logging and notification**
    - **Property 24: Data flow integrity**
    - **Property 25: Progress tracking**
    - **Property 26: Timeout and retry behavior**
    - **Property 27: Output persistence**
    - **Validates: Requirements 6.1, 6.2, 6.3, 6.4, 6.5, 6.6**

- [ ] 13. Implement error handling and user feedback system
  - [ ] 13.1 Create ErrorResponse model and error handling utilities
    - Implement ErrorResponse model with error codes and user messages
    - Create error code constants for each error category
    - Implement error message templates with suggested actions
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_
  
  - [ ]* 13.2 Write property tests for error handling
    - **Property 32: Upload failure messaging**
    - **Property 33: Low-confidence OCR warning**
    - **Property 34: Engine failure user-friendly messaging**
    - **Property 35: Translation/audio fallback**
    - **Validates: Requirements 8.1, 8.2, 8.3, 8.4**

- [ ] 14. Implement FastAPI REST API layer
  - [ ] 14.1 Create API endpoints for document processing
    - Implement POST /api/upload endpoint for document upload
    - Implement GET /api/status/{document_id} endpoint for progress tracking
    - Implement GET /api/result/{document_id} endpoint for retrieving results
    - Implement DELETE /api/document/{document_id} endpoint for user-initiated deletion
    - Add request validation using Pydantic models
    - _Requirements: 1.1, 1.2, 1.3, 6.4, 7.5_
  
  - [ ] 14.2 Add user preferences endpoint
    - Implement POST /api/preferences endpoint for setting language preferences
    - Implement GET /api/preferences endpoint for retrieving preferences
    - _Requirements: 4.1, 4.6_
  
  - [ ] 14.3 Configure TLS and security middleware
    - Configure TLS 1.3 for all API endpoints
    - Add CORS middleware with appropriate restrictions
    - Add rate limiting middleware
    - _Requirements: 7.1, 7.4_
  
  - [ ]* 14.4 Write integration tests for API endpoints
    - Test complete upload → process → retrieve flow
    - Test error responses for invalid inputs
    - Test concurrent request handling
    - _Requirements: 9.2_

- [ ] 15. Implement frontend UI components
  - [ ] 15.1 Create document upload interface
    - Build file upload component with drag-and-drop support
    - Add visual instructions and supported format indicators
    - Implement file validation on client side
    - _Requirements: 10.1_
  
  - [ ] 15.2 Create language selection interface
    - Build language selector dropdown with Hindi, Tamil, English options
    - Implement language preference persistence in session storage
    - Add interface localization based on selected language
    - _Requirements: 4.1, 4.6, 10.2_
  
  - [ ] 15.3 Create progress indicator component
    - Build progress bar showing pipeline stages
    - Display current stage and percentage complete
    - Add loading animations
    - _Requirements: 6.4, 9.6_
  
  - [ ] 15.4 Create summary display component
    - Build formatted summary display with sections, headings, and bullet points
    - Implement text highlighting synchronized with audio playback
    - Add adequate spacing and clear formatting (line-height ≥1.5)
    - _Requirements: 10.5, 5.4_
  
  - [ ] 15.5 Create audio playback controls
    - Build audio player with play, pause, replay, and download buttons
    - Implement audio-text synchronization for highlighting
    - Add large, accessible buttons (≥44x44px)
    - _Requirements: 5.3, 5.4, 5.5, 10.3_
  
  - [ ] 15.6 Add accessibility features
    - Use large text (≥16px) and clear buttons
    - Add visual icons alongside text labels
    - Implement responsive design for mobile devices (test at 320px, 768px, 1024px, 1920px)
    - _Requirements: 10.3, 10.4, 10.6_
  
  - [ ]* 15.7 Write property tests for UI components
    - **Property 38: Interface localization**
    - **Property 39: Accessibility element sizing**
    - **Property 40: Icon-text pairing**
    - **Property 41: Summary formatting**
    - **Property 42: Responsive design**
    - **Validates: Requirements 10.2, 10.3, 10.4, 10.5, 10.6**

- [ ] 16. Implement error display and help system
  - [ ] 16.1 Create error notification component
    - Build error message display with error code and user-friendly message
    - Add suggested actions list
    - Implement dismissible notifications
    - _Requirements: 8.1, 8.2, 8.3, 8.4_
  
  - [ ] 16.2 Create help section
    - Build help page with common issues and troubleshooting steps
    - Add FAQ section
    - _Requirements: 8.6_

- [ ] 17. Checkpoint - Ensure end-to-end flow works
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 18. Implement performance optimizations
  - [ ] 18.1 Add caching for RAG retrieval results
    - Implement Redis or in-memory cache for frequently retrieved medical context
    - Add cache invalidation strategy
    - _Requirements: 9.3_
  
  - [ ] 18.2 Optimize concurrent request handling
    - Configure FastAPI worker processes for concurrent processing
    - Add request queuing for load management
    - Test with 100 concurrent users
    - _Requirements: 9.2_
  
  - [ ]* 18.3 Write property tests for performance requirements
    - **Property 36: End-to-end processing time**
    - **Property 37: Component timing constraints**
    - **Validates: Requirements 9.1, 9.3, 9.4, 9.5**

- [ ] 19. Set up AWS Bedrock Knowledge Base with medical datasets
  - [ ] 19.1 Prepare medical datasets
    - Collect and format medical terminology dataset with plain-language definitions
    - Collect and format drug information dataset with purposes, dosages, side effects
    - Collect and format lab test reference dataset with normal ranges and interpretations
    - Generate synthetic medical texts for context examples
    - _Requirements: 2.1, 2.3, 2.4, 2.5_
  
  - [ ] 19.2 Configure AWS Bedrock Knowledge Base
    - Create knowledge base in AWS Bedrock
    - Upload datasets to S3 buckets
    - Configure knowledge base partitions for terminology, drugs, and lab tests
    - Set up embeddings and indexing
    - Test retrieval queries
    - _Requirements: 2.1, 2.3, 2.4, 2.5_

- [ ] 20. Implement data deletion scheduler
  - [ ] 20.1 Create background job for automatic deletion
    - Implement scheduled job that runs every hour
    - Query for documents with deletion_scheduled_at < current_time
    - Delete documents, summaries, and audio files
    - Log deletion actions
    - _Requirements: 7.3_
  
  - [ ]* 20.2 Write unit tests for deletion scheduler
    - Test deletion scheduling logic
    - Test deletion execution
    - Test that saved documents are not deleted
    - _Requirements: 7.3_

- [ ] 21. Final integration and deployment preparation
  - [ ] 21.1 Create deployment configuration
    - Write Dockerfile for containerization
    - Create docker-compose.yml for local development
    - Configure environment variables for production
    - Set up health check endpoints
    - _Requirements: All (deployment)_
  
  - [ ] 21.2 Create deployment documentation
    - Write README with setup instructions
    - Document API endpoints and request/response formats
    - Document environment variable configuration
    - Add troubleshooting guide
    - _Requirements: All (documentation)_
  
  - [ ]* 21.3 Run final integration tests
    - Test complete pipeline with various document types
    - Test error recovery scenarios
    - Test security features (encryption, deletion)
    - Test performance under load
    - _Requirements: All_

- [ ] 22. Final checkpoint - Complete system validation
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation at key milestones
- Property tests validate universal correctness properties across all inputs
- Unit tests validate specific examples, edge cases, and integration points
- The implementation follows a bottom-up approach: data models → agents → orchestration → API → frontend
- AWS Bedrock Knowledge Base setup (task 19) can be done in parallel with other tasks
- Frontend tasks (15-16) can be started after API layer is complete (task 14)
