# Backend Structure Documentation

## Overview
The backend of DocuGen is designed to be scalable, secure, and maintainable. It leverages Supabase for database operations, authentication support, and storage capabilities, along with custom API routes for specialized functionality like AI document generation.

## Technology Stack
- **Backend Framework**: Next.js API Routes + Supabase
- **Database**: PostgreSQL (via Supabase)
- **Authentication**: Clerk Auth (primary) with Supabase Auth integration
- **Storage**: Supabase Storage
- **Serverless Functions**: Vercel Functions (via Next.js API routes)
- **Payment Processing**: Stripe
- **AI Integration**: OpenAI API

## Architecture

### High-Level Architecture
```
┌────────────────┐     ┌────────────────┐     ┌────────────────┐
│   Next.js      │     │   Supabase     │     │    External    │
│  Application   │────▶│   Services     │────▶│     APIs       │
└────────────────┘     └────────────────┘     └────────────────┘
                               │                      │
                               ▼                      ▼
                        ┌────────────────┐     ┌────────────────┐
                        │  PostgreSQL    │     │    OpenAI      │
                        │   Database     │     │     API        │
                        └────────────────┘     └────────────────┘
```

### Database Schema
```
├── auth
│   ├── users
│   └── sessions
├── public
│   ├── profiles
│   ├── projects
│   ├── documents
│   ├── document_types
│   ├── document_templates
│   ├── tech_stacks
│   ├── technologies
│   ├── project_technologies
│   ├── subscriptions
│   └── usage_metrics
└── storage
    └── document_assets
```

## Entity Relationships

### Core Entities
1. **Users** - Managed by Clerk Auth with profile data in Supabase
2. **Projects** - Container for related documentation
3. **Documents** - Generated documentation of various types
4. **Document Types** - Templates and schemas for different document types
5. **Tech Stacks** - Predefined or custom technology combinations
6. **Technologies** - Individual technologies that can be part of a stack
7. **Subscriptions** - User subscription information from Stripe
8. **Usage Metrics** - Tracking usage for analytics and billing

### Key Relationships
- One User can have many Projects
- One Project can have many Documents
- Each Document belongs to a Document Type
- Each Project uses one Tech Stack
- Each Tech Stack contains multiple Technologies
- Each User can have one active Subscription

## API Structure

### Authentication & Users
```
/api/auth/webhook - Clerk webhook receiver
/api/users/profile - User profile operations
/api/users/preferences - User preference operations
```

### Projects
```
/api/projects - CRUD operations for projects
/api/projects/[id] - Specific project operations
/api/projects/[id]/documents - Documents within a project
```

### Documents
```
/api/documents - CRUD operations for documents
/api/documents/[id] - Specific document operations
/api/documents/generate - AI document generation endpoint
/api/documents/template/[type] - Get document template by type
```

### Tech Stacks
```
/api/tech-stacks - List available tech stacks
/api/tech-stacks/[id] - Specific tech stack operations
/api/technologies - List available technologies
```

### Billing & Subscriptions
```
/api/billing/create-checkout - Create Stripe checkout session
/api/billing/webhook - Stripe webhook receiver
/api/billing/subscriptions - User subscription operations
/api/billing/usage - User usage metrics
```

## Middleware
- **Authentication Middleware**: Validates user session and permissions
- **Rate Limiting Middleware**: Prevents API abuse
- **Logging Middleware**: Records API usage and errors
- **Error Handling Middleware**: Standardizes error responses

## Data Access Layer
The application uses a repository pattern to abstract database operations:

```typescript
// Example repository pattern
export class ProjectRepository {
  async getById(id: string): Promise<Project | null> {
    const { data, error } = await supabase
      .from('projects')
      .select('*')
      .eq('id', id)
      .single();
      
    if (error) throw new Error(error.message);
    return data;
  }
  
  async create(project: Omit<Project, 'id'>): Promise<Project> {
    const { data, error } = await supabase
      .from('projects')
      .insert(project)
      .select()
      .single();
      
    if (error) throw new Error(error.message);
    return data;
  }
  
  // Additional methods...
}
```

## OpenAI Integration
The OpenAI integration is managed through a service layer:

```typescript
export class DocumentGenerationService {
  private openai: OpenAIApi;
  
  constructor() {
    this.openai = new OpenAIApi(new Configuration({
      apiKey: process.env.OPENAI_API_KEY,
    }));
  }
  
  async generateDocument(
    projectDetails: ProjectDetails, 
    documentType: DocumentType
  ): Promise<string> {
    const prompt = this.buildPrompt(projectDetails, documentType);
    
    const response = await this.openai.createChatCompletion({
      model: "gpt-4",
      messages: [
        { role: "system", content: "You are a technical documentation expert." },
        { role: "user", content: prompt }
      ],
      temperature: 0.2,
    });
    
    return response.data.choices[0].message?.content || "";
  }
  
  private buildPrompt(
    projectDetails: ProjectDetails,
    documentType: DocumentType
  ): string {
    // Build detailed prompt based on document type and project details
    // ...
  }
}
```

## Stripe Integration
Payment processing is handled via a dedicated service:

```typescript
export class SubscriptionService {
  private stripe: Stripe;
  
  constructor() {
    this.stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
      apiVersion: '2023-10-16',
    });
  }
  
  async createCheckoutSession(
    userId: string,
    priceId: string
  ): Promise<string> {
    const session = await this.stripe.checkout.sessions.create({
      customer_email: user.email,
      line_items: [{ price: priceId, quantity: 1 }],
      mode: 'subscription',
      success_url: `${process.env.NEXT_PUBLIC_BASE_URL}/dashboard?success=true`,
      cancel_url: `${process.env.NEXT_PUBLIC_BASE_URL}/pricing`,
      metadata: { userId },
    });
    
    return session.url!;
  }
  
  // Additional methods for handling subscriptions, webhooks, etc.
}
```

## Security Considerations
1. **API Security**:
   - All endpoints authenticated except public-facing ones
   - Role-based access control for resources
   - Input validation on all endpoints

2. **Data Security**:
   - Encryption for sensitive data
   - Regular security audits
   - Proper environment variable management

3. **External API Security**:
   - Secure handling of API keys
   - Rate limiting for external API calls
   - Fallback mechanisms for API failures

## Error Handling Strategy
1. Standardized error responses
2. Detailed logging for debugging
3. Graceful degradation of functionality
4. User-friendly error messages

## Deployment Considerations
1. **Environment Configuration**:
   - Development, staging, and production environments
   - Environment-specific variables

2. **Database Migrations**:
   - Supabase migrations for schema changes
   - Data seeding for initial setup

3. **Monitoring**:
   - Logging integration (e.g., Vercel Logs)
   - Performance monitoring
   - Error tracking
