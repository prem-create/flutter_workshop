# AI Interview Coach - Design Document

## 1. System Architecture Overview

### 1.1 High-Level Architecture
The AI Interview Coach follows a **Clean Architecture** pattern with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Mobile    │  │   Tablet    │  │      Desktop        │  │
│  │     UI      │  │     UI      │  │        UI           │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│                    Business Logic Layer                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │    Home     │  │  Interview  │  │       MCQ           │  │
│  │    BLoC     │  │    BLoC     │  │      BLoC           │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Gemini    │  │    Local    │  │      Models         │  │
│  │ Repository  │  │   Storage   │  │   & Entities        │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│                    External Services                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Google    │  │   Future    │  │      Future         │  │
│  │  Gemini AI  │  │  Speech API │  │   Camera API        │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Design Principles
- **Single Responsibility:** Each component has one clear purpose
- **Dependency Inversion:** High-level modules don't depend on low-level modules
- **Open/Closed:** Open for extension, closed for modification
- **Responsive Design:** Adaptive UI for all screen sizes
- **State Management:** Centralized state using BLoC pattern

## 2. Application Structure

### 2.1 Directory Structure
```
lib/
├── core/
│   ├── constants/          # App-wide constants
│   ├── extensions/         # Dart extensions
│   ├── utils/             # Utility functions
│   └── widgets/           # Reusable widgets
├── pages/
│   ├── home_page/
│   │   ├── bloc/          # State management
│   │   └── ui/            # UI components
│   ├── camera_interview_page/
│   │   ├── bloc/
│   │   ├── models/
│   │   ├── repo/
│   │   └── ui/
│   ├── mcq_page/
│   │   ├── bloc/
│   │   ├── models/
│   │   ├── screens/
│   │   └── widgets/
│   └── talk_to_ai_page/
│       ├── bloc/
│       ├── repos/
│       └── ui/
├── routes/                # Navigation configuration
└── main.dart             # App entry point
```

### 2.2 Feature-Based Architecture
Each feature follows a consistent structure:
- **BLoC:** Business logic and state management
- **Models:** Data structures and entities
- **Repository:** Data access abstraction
- **UI:** User interface components (Mobile/Tablet/Desktop)

## 3. State Management Design

### 3.1 BLoC Pattern Implementation
```dart
// Event-driven architecture
Event → BLoC → State → UI Update

// Example: Interview Flow
StartInterviewEvent → CameraInterviewBloc → InterviewStartedState → UI Shows Interview
```

### 3.2 State Flow Diagram
```
┌─────────────┐    Event    ┌─────────────┐    State    ┌─────────────┐
│     UI      │ ──────────→ │    BLoC     │ ──────────→ │     UI      │
│ Components  │             │  Business   │             │  Updates    │
│             │ ←────────── │   Logic     │ ←────────── │             │
└─────────────┘   Listen    └─────────────┘   Emit      └─────────────┘
```

### 3.3 Key BLoCs
- **HomeBLoC:** Manages home screen state and navigation
- **CameraInterviewBLoC:** Handles interview session management
- **McqBLoC:** Controls quiz flow and scoring
- **TalkToAiBLoC:** Manages AI chat conversations

## 4. User Interface Design

### 4.1 Responsive Design Strategy
```dart
// Screen breakpoints
Mobile: < 768px
Tablet: 768px - 1024px
Desktop: > 1024px

// Implementation using responsive_builder
ScreenTypeLayout.builder(
  mobile: (context) => MobileView(),
  tablet: (context) => TabletView(),
  desktop: (context) => DesktopView(),
)
```

### 4.2 UI Component Hierarchy
```
MaterialApp
├── Router (go_router)
├── ScreenUtilInit (responsive scaling)
└── Pages
    ├── Home
    │   ├── MobileView
    │   ├── TabletView
    │   └── DesktopView
    ├── Interview
    │   ├── InitialUI
    │   ├── LoadingUI
    │   └── SuccessUI
    └── MCQ
        ├── QuizScreen
        └── ResultScreen
```

### 4.3 Design System
- **Colors:** Professional interview theme with accessibility compliance
- **Typography:** Clear, readable fonts optimized for different screen sizes
- **Spacing:** Consistent spacing using flutter_screenutil
- **Components:** Reusable widgets (buttons, cards, input fields)

## 5. Data Flow Architecture

### 5.1 AI Interview Flow
```
User Input → BLoC Event → Repository → Gemini API → Response Processing → State Update → UI Refresh
```

### 5.2 Conversation Management
```dart
// Memory-based conversation tracking
List<Map<String, dynamic>> _contents = [
  {
    "role": "user",
    "parts": [{"text": "User message"}]
  },
  {
    "role": "model", 
    "parts": [{"text": "AI response"}]
  }
];
```

### 5.3 Data Models
```dart
// Gemini Response Structure
Post
├── List<Candidate> candidates
    └── Candidate
        └── Content content
            ├── List<Part> parts
            └── String role

// Question Model
Question
├── String question
├── List<String> options
└── int correctAnswerIndex
```

## 6. API Integration Design

### 6.1 Gemini AI Integration
```dart
// Endpoint Configuration
Base URL: https://generativelanguage.googleapis.com/v1beta/
Model: gemini-3-flash-preview
Method: POST /generateContent

// Request Structure
{
  "contents": [
    {
      "role": "user|model",
      "parts": [{"text": "message"}]
    }
  ]
}
```

### 6.2 Repository Pattern
```dart
abstract class GeminiRepository {
  Future<String?> sendToGemini();
  void addCandidateAnswer(String answer);
  Future<String?> sendCandidateAnswer(String answer);
  void startInterview({...});
}
```

### 6.3 Error Handling Strategy
- **Network Errors:** Retry mechanism with exponential backoff
- **API Errors:** User-friendly error messages
- **Validation Errors:** Input sanitization and validation
- **State Errors:** Graceful state recovery

## 7. Security Design

### 7.1 API Security
- **API Key Management:** Secure storage of Gemini API keys
- **HTTPS Communication:** All API calls use secure protocols
- **Request Validation:** Input sanitization before API calls
- **Rate Limiting:** Respect API quotas and limits

### 7.2 Data Privacy
- **Local Storage:** Minimal data persistence
- **Session Management:** Temporary conversation storage
- **User Consent:** Clear privacy policies for AI analysis

## 8. Performance Optimization

### 8.1 UI Performance
- **Lazy Loading:** Load UI components on demand
- **Widget Optimization:** Efficient widget rebuilding
- **Image Optimization:** Compressed assets
- **Memory Management:** Proper disposal of resources

### 8.2 Network Performance
- **Request Optimization:** Minimize API calls
- **Caching Strategy:** Cache frequently used data
- **Compression:** Gzip compression for API responses
- **Connection Pooling:** Reuse HTTP connections

## 9. Testing Strategy

### 9.1 Testing Pyramid
```
┌─────────────────┐
│   E2E Tests     │  ← Integration & User Journey
├─────────────────┤
│ Widget Tests    │  ← UI Component Testing
├─────────────────┤
│  Unit Tests     │  ← Business Logic Testing
└─────────────────┘
```

### 9.2 Test Coverage Areas
- **BLoC Testing:** State transitions and business logic
- **Widget Testing:** UI component behavior
- **Repository Testing:** API integration and data handling
- **Model Testing:** Data serialization/deserialization

## 10. Deployment Architecture

### 10.1 Build Configuration
```yaml
# Platform-specific builds
Android: APK/AAB for Google Play Store
iOS: IPA for App Store
Web: PWA for web deployment
Desktop: Platform-specific executables
```

### 10.2 Environment Management
- **Development:** Local development with mock APIs
- **Staging:** Testing environment with real APIs
- **Production:** Live environment with monitoring

## 11. Future Architecture Considerations

### 11.1 Scalability Enhancements
- **Microservices:** Break down into smaller services
- **CDN Integration:** Global content delivery
- **Database Integration:** Persistent user data storage
- **Analytics Integration:** User behavior tracking

### 11.2 Feature Extensions
- **Speech Integration:** STT/TTS service architecture
- **Camera Analysis:** Computer vision pipeline
- **Multi-language:** Internationalization framework
- **Offline Mode:** Local AI model integration

### 11.3 Technology Evolution
- **AI Model Upgrades:** Support for newer AI models
- **Platform Updates:** Flutter and Dart version migrations
- **Third-party Integrations:** Additional service providers
- **Performance Monitoring:** Real-time performance tracking

## 12. Development Guidelines

### 12.1 Code Standards
- **Naming Conventions:** Clear, descriptive naming
- **Documentation:** Comprehensive code documentation
- **Code Review:** Peer review process
- **Version Control:** Git workflow with feature branches

### 12.2 Architecture Patterns
- **SOLID Principles:** Follow SOLID design principles
- **DRY Principle:** Don't repeat yourself
- **KISS Principle:** Keep it simple and straightforward
- **Clean Code:** Readable and maintainable code

---

**Document Version:** 1.0  
**Last Updated:** February 4, 2026  
**Authors:** Prem Dilliwar  
**Architecture Review:** Approved for Implementation