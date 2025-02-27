# Frontend Guidelines

## Overview
This document outlines the standards, best practices, and architectural patterns for the frontend development of DocuGen. Adhering to these guidelines will ensure consistency, maintainability, and scalability of the application.

## Technology Stack
- **Framework**: Next.js
- **UI Library**: React
- **Styling**: Tailwind CSS
- **Component Library**: shadcn/ui
- **Language**: TypeScript
- **State Management**: React Context API and/or Zustand
- **Form Handling**: React Hook Form with Zod validation

## Project Structure
```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/             # Authentication routes
│   ├── (dashboard)/        # Protected dashboard routes
│   ├── api/                # API routes
│   └── layout.tsx          # Root layout
├── components/             # Reusable components
│   ├── ui/                 # shadcn/ui components
│   ├── forms/              # Form components
│   ├── layouts/            # Layout components
│   └── documentation/      # Documentation-specific components
├── hooks/                  # Custom React hooks
├── lib/                    # Utility functions and libraries
├── styles/                 # Global styles and Tailwind configuration
├── types/                  # TypeScript type definitions
└── contexts/               # React context providers
```

## Component Guidelines

### Component Structure
1. Use functional components with hooks
2. Follow the component-per-file pattern
3. Implement proper TypeScript interfaces for props
4. Use shadcn/ui as the foundation for UI components

Example:
```tsx
import React from 'react';
import { Button } from '@/components/ui/button';

interface DocumentCardProps {
  title: string;
  description: string;
  documentType: string;
  onView: () => void;
  onEdit: () => void;
}

export const DocumentCard: React.FC<DocumentCardProps> = ({
  title,
  description,
  documentType,
  onView,
  onEdit
}) => {
  return (
    <div className="p-4 border rounded-lg shadow-sm">
      <h3 className="text-lg font-medium">{title}</h3>
      <p className="text-sm text-gray-500">{documentType}</p>
      <p className="mt-2 text-sm">{description}</p>
      <div className="mt-4 flex space-x-2">
        <Button variant="outline" size="sm" onClick={onView}>
          View
        </Button>
        <Button size="sm" onClick={onEdit}>
          Edit
        </Button>
      </div>
    </div>
  );
};
```

### Styling Approach
1. Use Tailwind CSS for all styling
2. Follow a consistent color scheme based on the design system
3. Utilize the shadcn/ui theme for consistent component styling
4. Create reusable utility classes for common patterns
5. Use CSS variables for theme configuration

### Responsive Design
1. Design mobile-first with Tailwind's responsive breakpoints
2. Test across multiple device sizes
3. Ensure accessibility on touch devices
4. Use fluid typography and spacing

## State Management
1. Use React Context API for global state that doesn't change frequently
2. Consider Zustand for more complex state management
3. Use React Query for server state management
4. Keep form state local using React Hook Form

## Performance Optimization
1. Implement code splitting via Next.js's dynamic imports
2. Use Next.js Image component for optimized images
3. Lazy load non-critical components
4. Memoize expensive computations and component renders
5. Implement proper loading states and skeletons

## Authentication
1. Implement Clerk Auth for authentication flows
2. Create protected routes using middleware
3. Handle authentication state globally
4. Provide clear user feedback for authentication actions

## Form Handling
1. Use React Hook Form for all forms
2. Implement Zod schemas for validation
3. Provide clear error messages and validation feedback
4. Use controlled components for complex form fields

## Accessibility Guidelines
1. Ensure proper ARIA attributes for custom components
2. Maintain keyboard navigation support
3. Follow proper heading hierarchy
4. Provide sufficient color contrast
5. Support screen readers

## Error Handling
1. Implement global error boundaries
2. Use toast notifications for user feedback
3. Log client-side errors
4. Provide helpful error recovery options

## Testing Strategy
1. Implement component tests using Vitest or Jest
2. Use React Testing Library for component testing
3. Implement end-to-end tests with Playwright or Cypress
4. Enforce testing of critical user flows

## Best Practices
1. Follow consistent naming conventions
2. Document complex components and functions
3. Use TypeScript strictly to avoid type errors
4. Optimize bundle size by avoiding unnecessary dependencies
5. Implement proper loading states for async operations
6. Use Next.js Server Components when appropriate
