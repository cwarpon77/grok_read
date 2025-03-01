# TheVAWorld Project Requirements

## Project Overview

TheVAWorld is a platform that connects employers with virtual assistants (VAs). The platform allows employers to find, hire, and manage VAs, while providing VAs with opportunities to showcase their skills and find employment.

## Technology Stack

- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: Supabase Auth
- **Hosting**: Vercel
- **Package Manager**: Yarn

## Core Features

### 1. User Authentication & Authorization

- User registration with email/password and social login options
- Role-based access control (Employer, VA, Admin)
- Profile creation and management
- Email verification and password reset

### 2. User Profiles

#### Employer Profiles
- Company information
- Project history
- Reviews and ratings
- Billing information
- Communication preferences

#### VA Profiles
- Skills and expertise
- Work history and portfolio
- Availability calendar
- Hourly rate and payment preferences
- Reviews and ratings

### 3. Job Marketplace

- Job posting creation and management
- Search and filter functionality for jobs
- Application submission and tracking
- Job recommendations based on skills and history

### 4. Messaging System

- Real-time chat between employers and VAs
- File sharing capabilities
- Message notifications
- Chat history

### 5. Time Tracking & Project Management

- Time tracking for hourly projects
- Task assignment and management
- Progress reporting
- Milestone tracking

### 6. Payment Processing

- Secure payment processing
- Multiple payment methods
- Escrow services for milestone-based projects
- Automatic invoicing
- Payment history

### 7. Review & Rating System

- Post-project reviews and ratings
- Detailed feedback options
- Rating aggregation and display

### 8. Admin Dashboard

- User management
- Content moderation
- Platform analytics
- Dispute resolution tools

## User Interface Requirements

### General UI

- Clean, professional design with black and white color scheme
- Responsive design for all device sizes
- Accessible design following WCAG guidelines
- Dark mode support

### Homepage

- Hero section with clear value proposition
- Featured VAs section
- How it works section
- Testimonials
- Call-to-action buttons

### Dashboard

- Overview of current projects/jobs
- Recent messages
- Upcoming deadlines
- Quick action buttons
- Analytics (earnings/spending)

### Search & Discovery

- Advanced search with multiple filters
- Skill-based matching
- Location-based search options
- Saved searches

## Technical Requirements

### Performance

- Page load time under 2 seconds
- Optimized image loading
- Code splitting and lazy loading
- Server-side rendering for critical pages

### Security

- HTTPS enforcement
- Data encryption
- CSRF protection
- Rate limiting
- Input validation
- Regular security audits

### Scalability

- Horizontal scaling capability
- Caching strategies
- Database optimization
- CDN integration

### Monitoring & Logging

- Error tracking
- Performance monitoring
- User behavior analytics
- Comprehensive logging

## Development Guidelines

### Code Quality

- TypeScript for type safety
- ESLint and Prettier for code formatting
- Unit and integration testing
- Code reviews for all PRs

### Deployment

- CI/CD pipeline
- Staging environment for testing
- Feature flags for gradual rollouts
- Automated testing in pipeline

### Documentation

- API documentation
- Component documentation
- Setup and deployment instructions
- User guides

## Phase 1 Implementation (MVP)

For the initial launch, focus on these core features:

1. User authentication and basic profiles
2. Simple job posting and application system
3. Basic messaging functionality
4. Simple payment processing
5. Minimal viable admin tools

## Future Enhancements (Post-MVP)

1. AI-powered matching algorithm
2. Mobile app development
3. Advanced analytics and reporting
4. Integration with popular project management tools
5. Skill assessment and certification
6. Learning resources and skill development
7. Community forums and networking

## Development Priorities

1. Set up project structure and core configuration
2. Implement authentication and user profiles
3. Develop job marketplace functionality
4. Create messaging system
5. Implement payment processing
6. Build review and rating system
7. Develop admin dashboard
8. Comprehensive testing and optimization
9. Deployment and monitoring setup

## Database Schema

### Tables and Relationships

#### `users` Table
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL CHECK (role IN ('employer', 'va', 'admin')),
    first_name TEXT NOT NULL,
    last_name TEXT NOT NULL,
    avatar_url TEXT,
    phone TEXT,
    country TEXT,
    timezone TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login TIMESTAMP WITH TIME ZONE,
    is_verified BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    email_notifications BOOLEAN DEFAULT TRUE
);
```

#### `employer_profiles` Table
```sql
CREATE TABLE employer_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    company_name TEXT,
    company_website TEXT,
    company_size TEXT,
    industry TEXT,
    company_description TEXT,
    company_logo_url TEXT,
    billing_address TEXT,
    billing_city TEXT,
    billing_state TEXT,
    billing_zip TEXT,
    billing_country TEXT,
    tax_id TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT unique_user_id UNIQUE (user_id)
);
```

#### `va_profiles` Table
```sql
CREATE TABLE va_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    headline TEXT,
    bio TEXT,
    hourly_rate DECIMAL(10, 2),
    years_experience INTEGER,
    education TEXT,
    languages TEXT[],
    availability TEXT,
    profile_visibility BOOLEAN DEFAULT TRUE,
    portfolio_url TEXT,
    resume_url TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT unique_user_id UNIQUE (user_id)
);
```

#### `skills` Table
```sql
CREATE TABLE skills (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT UNIQUE NOT NULL,
    category TEXT NOT NULL,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `va_skills` Table (Junction Table)
```sql
CREATE TABLE va_skills (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    va_profile_id UUID NOT NULL REFERENCES va_profiles(id) ON DELETE CASCADE,
    skill_id UUID NOT NULL REFERENCES skills(id) ON DELETE CASCADE,
    proficiency_level INTEGER NOT NULL CHECK (proficiency_level BETWEEN 1 AND 5),
    years_experience DECIMAL(4, 1),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT unique_va_skill UNIQUE (va_profile_id, skill_id)
);
```

#### `job_posts` Table
```sql
CREATE TABLE job_posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    employer_id UUID NOT NULL REFERENCES employer_profiles(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    description TEXT NOT NULL,
    job_type TEXT NOT NULL CHECK (job_type IN ('full-time', 'part-time', 'contract', 'one-time')),
    payment_type TEXT NOT NULL CHECK (payment_type IN ('hourly', 'fixed')),
    min_rate DECIMAL(10, 2),
    max_rate DECIMAL(10, 2),
    fixed_budget DECIMAL(10, 2),
    skills_required TEXT[],
    experience_level TEXT CHECK (experience_level IN ('entry', 'intermediate', 'expert')),
    estimated_duration TEXT,
    location_requirement TEXT,
    status TEXT NOT NULL DEFAULT 'open' CHECK (status IN ('draft', 'open', 'closed', 'filled', 'cancelled')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE
);
```

#### `job_applications` Table
```sql
CREATE TABLE job_applications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_post_id UUID NOT NULL REFERENCES job_posts(id) ON DELETE CASCADE,
    va_profile_id UUID NOT NULL REFERENCES va_profiles(id) ON DELETE CASCADE,
    cover_letter TEXT,
    proposed_rate DECIMAL(10, 2),
    availability_start_date DATE,
    status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'viewed', 'shortlisted', 'rejected', 'hired')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT unique_job_va UNIQUE (job_post_id, va_profile_id)
);
```

#### `contracts` Table
```sql
CREATE TABLE contracts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_post_id UUID REFERENCES job_posts(id),
    employer_id UUID NOT NULL REFERENCES employer_profiles(id) ON DELETE CASCADE,
    va_id UUID NOT NULL REFERENCES va_profiles(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    description TEXT,
    contract_type TEXT NOT NULL CHECK (contract_type IN ('hourly', 'fixed')),
    rate DECIMAL(10, 2),
    fixed_price DECIMAL(10, 2),
    start_date DATE,
    end_date DATE,
    payment_terms TEXT,
    status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'active', 'paused', 'completed', 'cancelled')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `milestones` Table
```sql
CREATE TABLE milestones (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_id UUID NOT NULL REFERENCES contracts(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    description TEXT,
    due_date DATE,
    amount DECIMAL(10, 2) NOT NULL,
    status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'in_progress', 'submitted', 'approved', 'rejected', 'paid')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `time_entries` Table
```sql
CREATE TABLE time_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_id UUID NOT NULL REFERENCES contracts(id) ON DELETE CASCADE,
    va_id UUID NOT NULL REFERENCES va_profiles(id) ON DELETE CASCADE,
    description TEXT,
    start_time TIMESTAMP WITH TIME ZONE NOT NULL,
    end_time TIMESTAMP WITH TIME ZONE,
    duration_minutes INTEGER,
    status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'approved', 'rejected', 'paid')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `conversations` Table
```sql
CREATE TABLE conversations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    employer_id UUID NOT NULL REFERENCES employer_profiles(id) ON DELETE CASCADE,
    va_id UUID NOT NULL REFERENCES va_profiles(id) ON DELETE CASCADE,
    job_post_id UUID REFERENCES job_posts(id),
    contract_id UUID REFERENCES contracts(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT unique_conversation UNIQUE (employer_id, va_id, COALESCE(job_post_id, '00000000-0000-0000-0000-000000000000'::UUID), COALESCE(contract_id, '00000000-0000-0000-0000-000000000000'::UUID))
);
```

#### `messages` Table
```sql
CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
    sender_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    attachment_url TEXT,
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `payments` Table
```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_id UUID NOT NULL REFERENCES contracts(id) ON DELETE CASCADE,
    milestone_id UUID REFERENCES milestones(id),
    payer_id UUID NOT NULL REFERENCES users(id),
    payee_id UUID NOT NULL REFERENCES users(id),
    amount DECIMAL(10, 2) NOT NULL,
    fee DECIMAL(10, 2) NOT NULL,
    payment_method TEXT,
    transaction_id TEXT,
    status TEXT NOT NULL CHECK (status IN ('pending', 'processing', 'completed', 'failed', 'refunded')),
    notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### `reviews` Table
```sql
CREATE TABLE reviews (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_id UUID NOT NULL REFERENCES contracts(id) ON DELETE CASCADE,
    reviewer_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    reviewee_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
    content TEXT,
    is_public BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT unique_review UNIQUE (contract_id, reviewer_id, reviewee_id)
);
```

#### `notifications` Table
```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    type TEXT NOT NULL,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    link TEXT,
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### Row-Level Security (RLS) Policies

#### Users Table RLS
```sql
-- Enable RLS on users table
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Users can view their own data
CREATE POLICY users_view_own ON users
    FOR SELECT
    USING (auth.uid() = id);

-- Users can update their own data
CREATE POLICY users_update_own ON users
    FOR UPDATE
    USING (auth.uid() = id);

-- Admins can view all users
CREATE POLICY admin_view_all_users ON users
    FOR SELECT
    USING (auth.jwt() ->> 'role' = 'admin');

-- Admins can update all users
CREATE POLICY admin_update_all_users ON users
    FOR UPDATE
    USING (auth.jwt() ->> 'role' = 'admin');
```

#### Employer Profiles RLS
```sql
-- Enable RLS on employer_profiles table
ALTER TABLE employer_profiles ENABLE ROW LEVEL SECURITY;

-- Employers can view and update their own profile
CREATE POLICY employer_manage_own_profile ON employer_profiles
    FOR ALL
    USING (auth.uid() = user_id);

-- VAs can view employer profiles
CREATE POLICY va_view_employer_profiles ON employer_profiles
    FOR SELECT
    USING (auth.jwt() ->> 'role' = 'va');

-- Admins can manage all employer profiles
CREATE POLICY admin_manage_all_employer_profiles ON employer_profiles
    FOR ALL
    USING (auth.jwt() ->> 'role' = 'admin');
```

#### VA Profiles RLS
```sql
-- Enable RLS on va_profiles table
ALTER TABLE va_profiles ENABLE ROW LEVEL SECURITY;

-- VAs can view and update their own profile
CREATE POLICY va_manage_own_profile ON va_profiles
    FOR ALL
    USING (auth.uid() = user_id);

-- Employers can view VA profiles
CREATE POLICY employer_view_va_profiles ON va_profiles
    FOR SELECT
    USING (auth.jwt() ->> 'role' = 'employer');

-- Admins can manage all VA profiles
CREATE POLICY admin_manage_all_va_profiles ON va_profiles
    FOR ALL
    USING (auth.jwt() ->> 'role' = 'admin');
```

#### Job Posts RLS
```sql
-- Enable RLS on job_posts table
ALTER TABLE job_posts ENABLE ROW LEVEL SECURITY;

-- Employers can manage their own job posts
CREATE POLICY employer_manage_own_jobs ON job_posts
    FOR ALL
    USING (auth.uid() IN (SELECT user_id FROM employer_profiles WHERE id = employer_id));

-- VAs can view open job posts
CREATE POLICY va_view_open_jobs ON job_posts
    FOR SELECT
    USING (status = 'open' AND auth.jwt() ->> 'role' = 'va');

-- Admins can manage all job posts
CREATE POLICY admin_manage_all_jobs ON job_posts
    FOR ALL
    USING (auth.jwt() ->> 'role' = 'admin');
```

#### Job Applications RLS
```sql
-- Enable RLS on job_applications table
ALTER TABLE job_applications ENABLE ROW LEVEL SECURITY;

-- VAs can manage their own applications
CREATE POLICY va_manage_own_applications ON job_applications
    FOR ALL
    USING (auth.uid() IN (SELECT user_id FROM va_profiles WHERE id = va_profile_id));

-- Employers can view applications for their job posts
CREATE POLICY employer_view_applications ON job_applications
    FOR SELECT
    USING (auth.uid() IN (
        SELECT ep.user_id 
        FROM employer_profiles ep
        JOIN job_posts jp ON ep.id = jp.employer_id
        WHERE jp.id = job_post_id
    ));

-- Admins can manage all applications
CREATE POLICY admin_manage_all_applications ON job_applications
    FOR ALL
    USING (auth.jwt() ->> 'role' = 'admin');
```

#### Contracts RLS
```sql
-- Enable RLS on contracts table
ALTER TABLE contracts ENABLE ROW LEVEL SECURITY;

-- Employers can view and manage contracts they're part of
CREATE POLICY employer_manage_own_contracts ON contracts
    FOR ALL
    USING (auth.uid() IN (SELECT user_id FROM employer_profiles WHERE id = employer_id));

-- VAs can view and manage contracts they're part of
CREATE POLICY va_manage_own_contracts ON contracts
    FOR ALL
    USING (auth.uid() IN (SELECT user_id FROM va_profiles WHERE id = va_id));

-- Admins can manage all contracts
CREATE POLICY admin_manage_all_contracts ON contracts
    FOR ALL
    USING (auth.jwt() ->> 'role' = 'admin');
```

#### Messages RLS
```sql
-- Enable RLS on messages table
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;

-- Users can view messages in conversations they're part of
CREATE POLICY view_own_conversation_messages ON messages
    FOR SELECT
    USING (
        auth.uid() IN (
            SELECT ep.user_id FROM conversations c
            JOIN employer_profiles ep ON c.employer_id = ep.id
            WHERE c.id = conversation_id
            UNION
            SELECT vp.user_id FROM conversations c
            JOIN va_profiles vp ON c.va_id = vp.id
            WHERE c.id = conversation_id
        )
    );

-- Users can send messages in conversations they're part of
CREATE POLICY send_messages_in_own_conversations ON messages
    FOR INSERT
    WITH CHECK (
        auth.uid() = sender_id AND
        auth.uid() IN (
            SELECT ep.user_id FROM conversations c
            JOIN employer_profiles ep ON c.employer_id = ep.id
            WHERE c.id = conversation_id
            UNION
            SELECT vp.user_id FROM conversations c
            JOIN va_profiles vp ON c.va_id = vp.id
            WHERE c.id = conversation_id
        )
    );

-- Admins can manage all messages
CREATE POLICY admin_manage_all_messages ON messages
    FOR ALL
    USING (auth.jwt() ->> 'role' = 'admin');
```

### Database Indexes

```sql
-- Users table indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);

-- Employer profiles indexes
CREATE INDEX idx_employer_profiles_user_id ON employer_profiles(user_id);
CREATE INDEX idx_employer_profiles_company_name ON employer_profiles(company_name);

-- VA profiles indexes
CREATE INDEX idx_va_profiles_user_id ON va_profiles(user_id);
CREATE INDEX idx_va_profiles_hourly_rate ON va_profiles(hourly_rate);

-- Skills indexes
CREATE INDEX idx_skills_category ON skills(category);

-- VA skills indexes
CREATE INDEX idx_va_skills_va_profile_id ON va_skills(va_profile_id);
CREATE INDEX idx_va_skills_skill_id ON va_skills(skill_id);

-- Job posts indexes
CREATE INDEX idx_job_posts_employer_id ON job_posts(employer_id);
CREATE INDEX idx_job_posts_status ON job_posts(status);
CREATE INDEX idx_job_posts_job_type ON job_posts(job_type);
CREATE INDEX idx_job_posts_created_at ON job_posts(created_at);

-- Job applications indexes
CREATE INDEX idx_job_applications_job_post_id ON job_applications(job_post_id);
CREATE INDEX idx_job_applications_va_profile_id ON job_applications(va_profile_id);
CREATE INDEX idx_job_applications_status ON job_applications(status);

-- Contracts indexes
CREATE INDEX idx_contracts_employer_id ON contracts(employer_id);
CREATE INDEX idx_contracts_va_id ON contracts(va_id);
CREATE INDEX idx_contracts_status ON contracts(status);

-- Messages indexes
CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX idx_messages_sender_id ON messages(sender_id);
CREATE INDEX idx_messages_created_at ON messages(created_at);

-- Payments indexes
CREATE INDEX idx_payments_contract_id ON payments(contract_id);
CREATE INDEX idx_payments_payer_id ON payments(payer_id);
CREATE INDEX idx_payments_payee_id ON payments(payee_id);
CREATE INDEX idx_payments_status ON payments(status);
```

### Database Triggers

```sql
-- Automatically update the updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create triggers for all tables with updated_at column
CREATE TRIGGER update_users_updated_at
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_employer_profiles_updated_at
BEFORE UPDATE ON employer_profiles
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_va_profiles_updated_at
BEFORE UPDATE ON va_profiles
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();

-- Similar triggers for all other tables with updated_at column
```

### Supabase Integration

For integrating with Supabase, the following considerations should be made:

1. **Authentication**: Utilize Supabase Auth for user management
   - Enable email/password authentication
   - Configure social login providers (Google, GitHub, etc.)
   - Set up email templates for verification and password reset

2. **Storage**: Use Supabase Storage for file uploads
   - Create buckets for different file types (avatars, resumes, attachments)
   - Set up appropriate access policies for each bucket

3. **Realtime**: Implement Supabase Realtime for messaging and notifications
   - Enable realtime subscriptions for messages table
   - Configure channels for different conversation types

4. **Edge Functions**: Use Supabase Edge Functions for serverless operations
   - Payment processing webhooks
   - Email notifications
   - Scheduled tasks (e.g., job expiration)

### Database Migration Strategy

For the initial setup and future migrations:

1. **Initial Schema**: Use Prisma to define and create the initial schema
2. **Migrations**: Generate and apply migrations using Prisma Migrate
3. **Seeding**: Create seed data for development and testing
4. **Backup**: Implement regular database backups

### Data Relationships Diagram

The database schema follows these key relationships:

- One User can have either one Employer Profile or one VA Profile
- Employers can create multiple Job Posts
- VAs can submit multiple Job Applications to different Job Posts
- When a Job Application is accepted, a Contract is created
- Contracts can have multiple Milestones or Time Entries
- Employers and VAs can have multiple Conversations
- Each Conversation contains multiple Messages
- Payments are linked to Contracts and optionally to Milestones
- Reviews are created after Contract completion

This comprehensive database schema provides a solid foundation for building the TheVAWorld platform, with proper security through Row-Level Security policies and optimized performance through strategic indexing.

## Environment Configuration

### Environment Variables

The project requires several environment variables for database connection and Supabase integration. These should be configured in the appropriate `.env` files:

#### `.env` (Base Configuration)

```
# Database connection
DATABASE_URL=postgresql://postgres:TheVAWorld%400603@db.sveqhuvprnvmanqzzhwf.supabase.co:5432/postgres

# Supabase configuration
NEXT_PUBLIC_SUPABASE_URL=https://sveqhuvprnvmanqzzhwf.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InN2ZXFodXZwcm52bWFucXp6aHdmIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDAzMjkwODQsImV4cCI6MjA1NTkwNTA4NH0.CEDos6IC2uy0Tr-9CgvZeiFTj_YCPrkJAcS6hcu_TWQ
```

#### `.env.local` (Local Development)

```
NEXT_PUBLIC_SUPABASE_URL=https://sveqhuvprnvmanqzzhwf.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InN2ZXFodXZwcm52bWFucXp6aHdmIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDAzMjkwODQsImV4cCI6MjA1NTkwNTA4NH0.CEDos6IC2uy0Tr-9CgvZeiFTj_YCPrkJAcS6hcu_TWQ
            
DATABASE_URL=postgresql://postgres:TheVAWorld%400603@db.sveqhuvprnvmanqzzhwf.supabase.co:5432/postgres
```

#### `.env.development` (Development Environment)

```
# Database connection
# Note: The @ in the password must be URL-encoded as %40
DATABASE_URL=postgresql://postgres:TheVAWorld%400603@db.sveqhuvprnvmanqzzhwf.supabase.co:5432/postgres

# Supabase configuration
NEXT_PUBLIC_SUPABASE_URL=https://sveqhuvprnvmanqzzhwf.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InN2ZXFodXZwcm52bWFucXp6aHdmIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDAzMjkwODQsImV4cCI6MjA1NTkwNTA4NH0.CEDos6IC2uy0Tr-9CgvZeiFTj_YCPrkJAcS6hcu_TWQ

# Next.js configuration
NODE_ENV=development
```

### Important Notes

1. **Security**: The environment files containing sensitive information should never be committed to version control. Add them to `.gitignore`.

2. **Supabase Anon Key**: The same anonymous key is used across all environments:
   ```
   eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InN2ZXFodXZwcm52bWFucXp6aHdmIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDAzMjkwODQsImV4cCI6MjA1NTkwNTA4NH0.CEDos6IC2uy0Tr-9CgvZeiFTj_YCPrkJAcS6hcu_TWQ
   ```

3. **URL Encoding**: Note that special characters in the database password (like `@`) must be URL-encoded as `%40`.

4. **Environment Validation**: Implement validation for required environment variables at application startup to prevent runtime errors.

## Server Configuration Best Practices

Based on our previous troubleshooting experience, here are key best practices for server configuration:

### Next.js Development Server

1. **Always use the `--hostname 0.0.0.0` flag** when starting the development server to allow access from all network interfaces:
   ```bash
   yarn dev --hostname 0.0.0.0
   ```

2. **Avoid custom server implementations** unless absolutely necessary. The built-in Next.js development server is optimized for most use cases.

3. **If a custom server is needed**, keep it as simple as possible:
   ```javascript
   // server.mjs - Minimal custom server example
   import { createServer } from 'http';
   import { parse } from 'url';
   import next from 'next';
   
   const dev = process.env.NODE_ENV !== 'production';
   const hostname = '0.0.0.0';
   const port = parseInt(process.env.PORT || '3000', 10);
   
   const app = next({ dev, hostname, port });
   const handle = app.getRequestHandler();
   
   app.prepare().then(() => {
     createServer((req, res) => {
       const parsedUrl = parse(req.url, true);
       handle(req, res, parsedUrl);
     }).listen(port, hostname, (err) => {
       if (err) throw err;
       console.log(`> Ready on http://${hostname}:${port}`);
     });
   });
   ```

4. **Avoid middleware complexity** during initial development. Add middleware incrementally and test after each addition.

5. **Monitor for hanging processes** and implement proper error handling and timeouts.

### Module System Consistency

1. **Choose one module system** (ESM or CommonJS) and stick with it consistently throughout the project.

2. **For ESM**:
   - Use `"type": "module"` in package.json
   - Use `.mjs` extension for ESM files
   - Use import/export syntax

3. **For CommonJS**:
   - Use `require()` and `module.exports`
   - Use `.cjs` extension for CommonJS files in an ESM project

4. **Avoid mixing module systems** within the same file or closely related files.

### Configuration Simplicity

1. **Start with minimal configuration** and add features incrementally.

2. **In next.config.js**, avoid experimental features in production:
   ```javascript
   /** @type {import('next').NextConfig} */
   const nextConfig = {
     reactStrictMode: true,
     images: {
       domains: ['sveqhuvprnvmanqzzhwf.supabase.co'],
     },
     // Add other non-experimental configurations as needed
   };
   
   module.exports = nextConfig;
   ```

3. **Test configuration changes** in isolation to identify potential issues.

4. **Document all configuration decisions** and their rationale.

By following these best practices, the rebuilt project should avoid the connectivity issues experienced in the previous implementation.

## Project Structure

```
thevaworld/
├── .env                  # Base environment variables
├── .env.local            # Local development environment variables
├── .env.development      # Development environment variables
├── .gitignore            # Git ignore file
├── next.config.js        # Next.js configuration
├── package.json          # Project dependencies and scripts
├── tsconfig.json         # TypeScript configuration
├── postcss.config.js     # PostCSS configuration for Tailwind
├── tailwind.config.js    # Tailwind CSS configuration
├── prisma/               # Prisma ORM files
│   ├── schema.prisma     # Prisma schema
│   └── migrations/       # Database migrations
├── public/               # Static assets
├── src/                  # Source code
│   ├── app/              # Next.js App Router
│   │   ├── api/          # API routes
│   │   ├── auth/         # Authentication pages
│   │   ├── dashboard/    # Dashboard pages
│   │   ├── jobs/         # Job marketplace pages
│   │   ├── messages/     # Messaging system pages
│   │   ├── profile/      # Profile pages
│   │   ├── admin/        # Admin dashboard
│   │   └── page.tsx      # Homepage
│   ├── components/       # Reusable UI components
│   │   ├── ui/           # Basic UI components
│   │   ├── forms/        # Form components
│   │   ├── layout/       # Layout components
│   │   └── shared/       # Shared components
│   ├── lib/              # Utility functions and libraries
│   │   ├── supabase/     # Supabase client and utilities
│   │   ├── prisma/       # Prisma client and utilities
│   │   ├── auth/         # Authentication utilities
│   │   └── utils/        # General utilities
│   ├── hooks/            # Custom React hooks
│   ├── types/            # TypeScript type definitions
│   ├── styles/           # Global styles
│   └── middleware.ts     # Next.js middleware
└── tests/                # Test files
```

## Development Setup Instructions

### Prerequisites

- Node.js v18+ (LTS recommended)
- Yarn package manager
- PostgreSQL (or Supabase account)
- Git

### Initial Setup

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd thevaworld
   ```

2. **Install dependencies**:
   ```bash
   yarn install
   ```

3. **Set up environment variables**:
   - Copy `.env.example` to `.env.local`
   - Fill in the required environment variables

4. **Initialize the database**:
   ```bash
   npx prisma migrate dev --name init
   ```

5. **Start the development server**:
   ```bash
   yarn dev --hostname 0.0.0.0
   ```

### Development Workflow

1. **Create a new feature branch**:
   ```bash
   git checkout -b feature/feature-name
   ```

2. **Make changes and test locally**

3. **Run linting and type checking**:
   ```bash
   yarn lint
   yarn type-check
   ```

4. **Commit changes with descriptive messages**:
   ```bash
   git commit -m "Add feature: description of changes"
   ```

5. **Push changes and create a pull request**:
   ```bash
   git push origin feature/feature-name
   ```

## Deployment Strategy

### Vercel Deployment

1. **Connect repository to Vercel**:
   - Link GitHub/GitLab repository to Vercel project

2. **Configure environment variables**:
   - Add all required environment variables in Vercel dashboard

3. **Configure build settings**:
   - Build command: `yarn build`
   - Output directory: `.next`

4. **Set up preview deployments**:
   - Enable preview deployments for pull requests

### CI/CD Pipeline

1. **GitHub Actions workflow**:
   - Run tests on pull requests
   - Check code quality and formatting
   - Deploy to staging environment for review

2. **Production deployment**:
   - Automatic deployment on merge to main branch
   - Manual promotion to production if needed

## Performance Optimization Strategies

1. **Image Optimization**:
   - Use Next.js Image component for automatic optimization
   - Implement responsive images for different device sizes
   - Use WebP format where supported

2. **Code Splitting**:
   - Implement dynamic imports for large components
   - Use React.lazy for component lazy loading
   - Configure webpack chunks appropriately

3. **Caching Strategy**:
   - Implement SWR for data fetching with caching
   - Use Incremental Static Regeneration for semi-static pages
   - Configure appropriate cache headers

4. **Bundle Size Optimization**:
   - Regular analysis of bundle size with tools like `next-bundle-analyzer`
   - Tree-shaking unused code
   - Optimizing third-party dependencies

## Accessibility Guidelines

1. **WCAG Compliance**:
   - Target WCAG 2.1 Level AA compliance
   - Implement proper semantic HTML
   - Ensure keyboard navigation works throughout the application

2. **Screen Reader Support**:
   - Add appropriate ARIA attributes
   - Test with screen readers (NVDA, VoiceOver)
   - Implement skip links for navigation

3. **Color Contrast**:
   - Ensure sufficient contrast ratios for text
   - Don't rely solely on color to convey information
   - Provide visual indicators for interactive elements

4. **Responsive Design**:
   - Ensure the application is usable at various zoom levels
   - Support text resizing without breaking layouts
   - Implement responsive designs for all device sizes

## Error Handling and Monitoring

1. **Client-Side Error Handling**:
   - Implement global error boundaries
   - Provide user-friendly error messages
   - Add retry mechanisms for failed requests

2. **Server-Side Error Handling**:
   - Implement structured error responses
   - Log detailed error information
   - Handle edge cases gracefully

3. **Monitoring Tools**:
   - Implement Sentry for error tracking
   - Use Vercel Analytics for performance monitoring
   - Set up logging infrastructure

4. **Alerting System**:
   - Configure alerts for critical errors
   - Set up uptime monitoring
   - Implement performance degradation alerts

## Security Considerations

1. **Authentication Security**:
   - Implement proper session management
   - Use secure cookies with HttpOnly and SameSite flags
   - Implement CSRF protection

2. **Data Protection**:
   - Encrypt sensitive data at rest
   - Implement proper input validation
   - Use parameterized queries for database access

3. **API Security**:
   - Implement rate limiting
   - Use proper authentication for all API endpoints
   - Validate and sanitize all inputs

4. **Frontend Security**:
   - Implement Content Security Policy
   - Use Subresource Integrity for external resources
   - Protect against XSS with proper output encoding

## Maintenance and Support Plan

1. **Regular Updates**:
   - Schedule regular dependency updates
   - Plan for Next.js version upgrades
   - Monitor for security advisories

2. **Backup Strategy**:
   - Implement automated database backups
   - Store backups in secure, redundant storage
   - Test backup restoration regularly

3. **Documentation**:
   - Maintain up-to-date technical documentation
   - Document common issues and solutions
   - Create user guides for platform features

4. **Support Process**:
   - Define support tiers and response times
   - Implement a ticketing system for issue tracking
   - Create a knowledge base for self-service support

This comprehensive requirements document now provides a complete roadmap for rebuilding TheVAWorld project, addressing all aspects from technical architecture to deployment and maintenance.
