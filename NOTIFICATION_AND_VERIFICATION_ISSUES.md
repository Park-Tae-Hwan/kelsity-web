# Notification and Verification Issues Analysis

## Issue 1: /api/notifications returning 404

### Root Cause
The backend does not have a REST API endpoint for `/api/notifications`. The notification system is currently implemented only through Socket.IO real-time events.

### Current Implementation
1. **Backend**: Notifications are handled via Socket.IO in `/src/socket/notificationHandlers.ts`
   - Only responds to socket events: `notifications:get`, `notifications:subscribe`
   - Returns empty notifications list by default
   
2. **Frontend**: The `NotificationService` tries to fetch from `/api/notifications` REST endpoint
   - Falls back to Socket.IO when REST API fails
   - Line 161-189 in `lib/services/notification/notification_service.dart`

### Solution Needed
Either:
1. **Option A**: Create REST API endpoints for notifications in the backend
   - Create `/src/routes/notifications.ts`
   - Add CRUD operations for notifications
   - Register routes in `/src/index.ts`

2. **Option B**: Update frontend to rely solely on Socket.IO for notifications
   - Remove REST API calls from `NotificationService`
   - Use only Socket.IO events

## Issue 2: verify-intent failing with "Invalid or expired verification token"

### Root Cause
The verification flow is working correctly, but there may be issues with:
1. Token expiration (24 hours)
2. Token already used
3. URL encoding issues with the token

### Current Implementation
1. **Email Service**: Sends verification email with URL:
   - URL format: `https://kelsity.com/verify-email-v2?token={token}`
   - Token is a 32-byte hex string

2. **Frontend**: 
   - Routes to `VerifyEmailHandler` which calls `verifyIntent(token)`
   - API call: `GET /api/auth-verify-first/verify-intent/{token}`

3. **Backend**:
   - Checks if token exists in `registrationIntent` table
   - Validates expiration time
   - Returns email and token if valid

### Potential Issues
1. **Token Format**: Ensure the token isn't being URL-encoded twice
2. **Database Check**: The token might not exist or already be deleted
3. **Timing**: User might be clicking expired links

## Issue 3: Post-Registration Flow

### What Happens After Registration
1. **Successful Registration Flow**:
   - User completes registration in `PasswordSetupScreen`
   - Auth tokens are stored in secure storage
   - User is redirected to `MainNavigation`
   - `MainNavigation` loads `HomeScreen` by default
   - `HomeScreen` initializes services including `NotificationService`

2. **First Load Issues**:
   - `NotificationService` tries to fetch from non-existent `/api/notifications`
   - This fails but is handled gracefully with Socket.IO fallback
   - User sees the home dashboard

## Recommendations

### Immediate Fixes
1. **For Notifications**: Implement Option B (Socket.IO only) as it's simpler
2. **For Verification**: Add better error logging to identify exact failure point
3. **For User Experience**: Show loading states during service initialization

### Long-term Improvements
1. Implement proper REST API for notifications
2. Add notification persistence in database
3. Improve error messages for expired/invalid tokens
4. Add token refresh mechanism for verification emails