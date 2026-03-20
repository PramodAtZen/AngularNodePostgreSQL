# Data Model: Real Estate Management System

## Entity: User
- Purpose: Represents authenticated platform users and role assignment.
- Fields:
  - id (UUID, primary key)
  - email (string, unique, required)
  - password_hash (string, required)
  - full_name (string, required)
  - role (enum: AGENT_VENDOR, ADMIN_MANAGER, END_USER_BUYER, SYSTEM_ADMIN)
  - is_active (boolean, default true)
  - created_at (timestamp)
  - updated_at (timestamp)
- Validation Rules:
  - email must be valid format and unique.
  - role must be one of allowed enum values.

## Entity: Property
- Purpose: Canonical listing entity for lifecycle and discovery.
- Fields:
  - id (UUID, primary key)
  - owner_user_id (UUID, foreign key -> User.id)
  - title (string, required)
  - description (text, required)
  - price (numeric, required, > 0)
  - property_type (string, required)
  - location_city (string, required)
  - location_state (string, required)
  - location_country (string, required)
  - address_line (string, required)
  - postal_code (string, required)
  - amenities (JSONB, default {})
  - status (enum: DRAFT, PENDING_APPROVAL, CHANGES_REQUESTED, PUBLISHED, SOLD, ARCHIVED)
  - published_at (timestamp, nullable)
  - sold_or_archived_at (timestamp, nullable)
  - created_at (timestamp)
  - updated_at (timestamp)
- Validation Rules:
  - status transitions must follow workflow rules.
  - amenities JSONB must be valid JSON object.
  - owner cannot approve their own property.

## Entity: PropertySubmission
- Purpose: Versioned multi-page onboarding snapshot used during approval cycles.
- Fields:
  - id (UUID, primary key)
  - property_id (UUID, foreign key -> Property.id)
  - submission_version (integer, required)
  - page_payload (JSONB, required)
  - submitted_by_user_id (UUID, foreign key -> User.id)
  - submitted_at (timestamp, nullable)
  - created_at (timestamp)
- Validation Rules:
  - submission_version must increment per property.
  - submitted_at required when moving to PENDING_APPROVAL.

## Entity: ApprovalStage
- Purpose: Defines required approval chain for a property submission.
- Fields:
  - id (UUID, primary key)
  - property_submission_id (UUID, foreign key -> PropertySubmission.id)
  - stage_order (integer, required)
  - approver_role (enum: ADMIN_MANAGER, SYSTEM_ADMIN)
  - status (enum: PENDING, APPROVED, CHANGES_REQUESTED)
  - acted_by_user_id (UUID, foreign key -> User.id, nullable)
  - acted_at (timestamp, nullable)
  - comments (text, nullable)
- Validation Rules:
  - stage_order must be unique within a submission.
  - acted_by_user_id role must satisfy approver_role.

## Entity: Lead
- Purpose: Buyer inquiry submitted against a published property.
- Fields:
  - id (UUID, primary key)
  - property_id (UUID, foreign key -> Property.id)
  - buyer_user_id (UUID, foreign key -> User.id, nullable for guest lead)
  - buyer_name (string, required)
  - buyer_email (string, required)
  - buyer_phone (string, nullable)
  - message (text, required)
  - status (enum: NEW, CONTACTED, QUALIFIED, CLOSED)
  - created_at (timestamp)
  - updated_at (timestamp)
- Validation Rules:
  - property must be in PUBLISHED status when lead is created.
  - buyer_email must be valid format.

## Entity: ListingAnalyticsEvent
- Purpose: Event stream for views and inquiry interactions.
- Fields:
  - id (UUID, primary key)
  - property_id (UUID, foreign key -> Property.id)
  - event_type (enum: VIEW, INQUIRY)
  - actor_user_id (UUID, foreign key -> User.id, nullable)
  - occurred_at (timestamp)
  - metadata (JSONB, default {})
- Validation Rules:
  - event_type required and constrained to enum.
  - occurred_at required for ordering and aggregation.

## Entity: AuditLog
- Purpose: Compliance-grade immutable record for sensitive actions.
- Fields:
  - id (UUID, primary key)
  - actor_user_id (UUID, foreign key -> User.id)
  - entity_type (string, required)
  - entity_id (UUID, required)
  - action (string, required)
  - previous_state (JSONB, nullable)
  - next_state (JSONB, nullable)
  - reason (text, nullable)
  - occurred_at (timestamp)
- Validation Rules:
  - action and occurred_at required.
  - log entries are append-only.

## State Transitions: Property.status
- DRAFT -> PENDING_APPROVAL (submit for review)
- PENDING_APPROVAL -> CHANGES_REQUESTED (review feedback)
- CHANGES_REQUESTED -> PENDING_APPROVAL (resubmission)
- PENDING_APPROVAL -> PUBLISHED (all stages approved)
- PUBLISHED -> SOLD (transaction closure)
- PUBLISHED -> ARCHIVED (withdrawn or inactive)
- ARCHIVED -> PUBLISHED (optional relist, policy-controlled)

## Relationship Summary
- User (1) -> (many) Property by owner_user_id
- Property (1) -> (many) PropertySubmission
- PropertySubmission (1) -> (many) ApprovalStage
- Property (1) -> (many) Lead
- Property (1) -> (many) ListingAnalyticsEvent
- User (1) -> (many) AuditLog as actor
