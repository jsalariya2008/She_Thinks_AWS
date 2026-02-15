# ELI5 Learning Tool - Design Document

## System Architecture

### High-Level Architecture
```
[Browser] <--> [Java Backend] <--> [MySQL Database]
                    |
                    v
              [Gemini AI API]
```

### Architecture Pattern
- **Pattern**: Model-View-Controller (MVC)
- **Frontend**: Server-side rendered HTML pages
- **Backend**: Java Servlets or Spring Boot
- **Database**: MySQL with JDBC connection

## Technology Stack

### Frontend
- **HTML5**: Semantic markup for forms and content
- **CSS3**: Styling with focus on simplicity and readability
- **No JavaScript**: Pure server-side rendering with form submissions

### Backend
- **Java 17+**: Core programming language
- **Framework Options**:
  - Option A: Spring Boot (recommended for easier setup)
  - Option B: Java Servlets with Tomcat
- **HTTP Client**: For Gemini API integration
- **JDBC**: Database connectivity

### Database
- **MySQL 8.0+**: Relational database
- **Connection Pooling**: HikariCP or built-in Spring Boot pooling

### External Services
- **Gemini AI API**: For generating ELI5 explanations

## Component Design

### 1. Frontend Components

#### Home Page (`index.html`)
- Header with project title
- Topic submission form
- Link to history page
- Simple, centered layout

#### History Page (`history.html`)
- List of all previous topics and explanations
- Back to home link
- Chronological display

#### Result Page (`result.html`)
- Display submitted topic
- Show generated ELI5 explanation
- Options: Submit another topic, View history

### 2. Backend Components

#### Servlets/Controllers

**TopicController**
- `GET /`: Serve home page
- `POST /submit`: Handle topic submission
  - Validate input
  - Call AI service
  - Save to database
  - Redirect to result page
- `GET /result`: Display explanation result
- `GET /history`: Retrieve and display all explanations

#### Services

**AIService**
- `generateExplanation(String topic)`: Call Gemini API
- Handle API authentication
- Parse API response
- Error handling and retries

**TopicService**
- `saveTopic(Topic topic)`: Save topic and explanation to database
- `getAllTopics()`: Retrieve all topics with explanations
- `getTopicById(Long id)`: Retrieve specific topic

#### Models

**Topic**
```java
class Topic {
    Long id;
    String topicText;
    String explanation;
    LocalDateTime createdAt;
}
```

#### Repository/DAO

**TopicRepository**
- CRUD operations for topics
- Database connection management
- SQL query execution

### 3. Database Design

#### Schema

**topics table**
```sql
CREATE TABLE topics (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    topic_text VARCHAR(500) NOT NULL,
    explanation TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_created_at (created_at)
);
```

## API Integration Design

### Gemini AI API Integration

**Endpoint**: Google Gemini API
**Authentication**: API Key (stored in environment variables)

**Request Format**:
```
POST https://generativelanguage.googleapis.com/v1/models/gemini-pro:generateContent
Headers:
  - Content-Type: application/json
  - x-goog-api-key: [API_KEY]

Body:
{
  "contents": [{
    "parts": [{
      "text": "Explain like I'm 5: [USER_TOPIC]. Use simple analogies and stories."
    }]
  }]
}
```

**Response Handling**:
- Extract explanation text from response
- Handle rate limiting (429 errors)
- Implement retry logic with exponential backoff
- Fallback error message if API fails

## User Flow

### Submit Topic Flow
1. User visits home page
2. User enters topic in form
3. User clicks "Explain" button
4. Form submits via POST to `/submit`
5. Backend validates input
6. Backend calls Gemini API with prompt
7. Backend saves topic + explanation to database
8. Backend redirects to `/result` with topic ID
9. Result page displays the explanation

### View History Flow
1. User clicks "View History" link
2. Browser navigates to `/history`
3. Backend retrieves all topics from database
4. Backend renders history page with all explanations
5. User can click back to home

## UI/UX Design

### Design Principles
- **Minimalism**: Clean, distraction-free interface
- **Readability**: Large, clear fonts
- **Accessibility**: Semantic HTML, proper contrast ratios
- **Simplicity**: No complex interactions

### Color Scheme
- Primary: #4A90E2 (Blue - trust and learning)
- Background: #FFFFFF (White - clean)
- Text: #333333 (Dark gray - readable)
- Accent: #50C878 (Green - success states)

### Typography
- Headings: Sans-serif, 24-32px
- Body: Sans-serif, 16-18px
- Line height: 1.6 for readability

### Layout
- Centered content with max-width: 800px
- Generous padding and margins
- Card-based design for explanations
- Mobile-responsive using CSS media queries

## Configuration Management

### Environment Variables
```
DB_HOST=localhost
DB_PORT=3306
DB_NAME=eli5_db
DB_USER=eli5_user
DB_PASSWORD=[secure_password]
GEMINI_API_KEY=[api_key]
```

### Application Properties (Spring Boot)
```properties
spring.datasource.url=jdbc:mysql://${DB_HOST}:${DB_PORT}/${DB_NAME}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
gemini.api.key=${GEMINI_API_KEY}
gemini.api.url=https://generativelanguage.googleapis.com/v1/models/gemini-pro:generateContent
```

## Error Handling Strategy

### User-Facing Errors
- Empty topic submission: "Please enter a topic to explain"
- API failure: "Sorry, we couldn't generate an explanation right now. Please try again."
- Database error: "Something went wrong. Please try again later."

### Logging
- Log all API calls and responses
- Log database errors with stack traces
- Log user submissions (without PII)

## Security Considerations

### Input Validation
- Sanitize all user inputs
- Limit topic length (max 500 characters)
- Prevent SQL injection using prepared statements

### API Security
- Store API keys in environment variables
- Never expose keys in frontend code
- Implement rate limiting on backend

### Database Security
- Use parameterized queries
- Principle of least privilege for database user
- Regular backups

## Deployment Architecture

### Development Environment
- Local MySQL instance
- Java application running on localhost:8080
- Environment variables in `.env` file

### Production Environment (Future)
- Cloud hosting (AWS/GCP/Azure)
- Managed MySQL database
- HTTPS with SSL certificate
- Environment variables in cloud configuration

## Testing Strategy

### Unit Tests
- Test AIService API integration
- Test TopicService business logic
- Test input validation

### Integration Tests
- Test database operations
- Test end-to-end topic submission flow

### Manual Testing
- Test all user flows in browser
- Test error scenarios
- Test on different screen sizes

## Performance Optimization

### Database
- Index on `created_at` for history queries
- Connection pooling to reduce overhead
- Limit history results (e.g., last 100 entries)

### API Calls
- Cache common explanations (future enhancement)
- Implement timeout for API calls (10 seconds)
- Async processing for better responsiveness (future)

## Future Enhancements

### Phase 2 Features
- Add JavaScript for better UX
- Real-time explanation generation feedback
- Search and filter history
- User accounts and authentication

### Phase 3 Features
- Topic categories (AI, ML, Cybersecurity)
- Difficulty levels (ELI5, ELI10, ELI15)
- Community voting on best explanations
- Export and share functionality
