# Kelsity Signup Flow Implementation Guide

## Overview
This guide provides technical implementation details for the Kelsity signup process with full legal compliance.

## 1. Database Schema for Consent Management

```sql
-- Users table extension
ALTER TABLE users ADD COLUMN age_bracket VARCHAR(20); -- 'under_13', '13_17', '18_plus'
ALTER TABLE users ADD COLUMN parental_consent_required BOOLEAN DEFAULT FALSE;
ALTER TABLE users ADD COLUMN parental_email VARCHAR(255);

-- Consent records table
CREATE TABLE consent_records (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    consent_version VARCHAR(10),
    terms_version VARCHAR(10),
    privacy_version VARCHAR(10),
    created_at TIMESTAMP,
    ip_address INET,
    user_agent TEXT,
    consent_details JSONB
);

-- Consent types table
CREATE TABLE consent_types (
    id UUID PRIMARY KEY,
    consent_key VARCHAR(50) UNIQUE,
    description TEXT,
    is_required BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP
);

-- User consents table
CREATE TABLE user_consents (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    consent_type_id UUID REFERENCES consent_types(id),
    granted BOOLEAN DEFAULT FALSE,
    granted_at TIMESTAMP,
    revoked_at TIMESTAMP,
    version VARCHAR(10)
);

-- Marketing preferences table
CREATE TABLE marketing_preferences (
    user_id UUID PRIMARY KEY REFERENCES users(id),
    email_updates BOOLEAN DEFAULT FALSE,
    email_promotions BOOLEAN DEFAULT FALSE,
    email_tips BOOLEAN DEFAULT FALSE,
    push_reminders BOOLEAN DEFAULT TRUE,
    push_updates BOOLEAN DEFAULT FALSE,
    push_promotions BOOLEAN DEFAULT FALSE,
    updated_at TIMESTAMP
);
```

## 2. Age Gate Implementation

```typescript
// AgeGate.tsx
interface AgeGateProps {
  onAgeSelected: (ageBracket: AgeBracket) => void;
}

type AgeBracket = 'under_13' | '13_17' | '18_plus';

const AgeGate: React.FC<AgeGateProps> = ({ onAgeSelected }) => {
  return (
    <div className="age-gate-container">
      <h2>Welcome to Kelsity!</h2>
      <p>To get started, please tell us your age:</p>
      
      <div className="age-options">
        <button 
          onClick={() => handleAgeSelection('under_13')}
          className="age-option"
        >
          I am under 13 years old
        </button>
        
        <button 
          onClick={() => handleAgeSelection('13_17')}
          className="age-option"
        >
          I am 13-17 years old
        </button>
        
        <button 
          onClick={() => handleAgeSelection('18_plus')}
          className="age-option"
        >
          I am 18 or older
        </button>
      </div>
      
      <p className="privacy-note">
        We ask for your age to comply with privacy laws and provide 
        age-appropriate features.
      </p>
    </div>
  );
};

// Handle under 13 rejection
const Under13Redirect: React.FC = () => {
  return (
    <div className="age-restriction">
      <h2>Sorry!</h2>
      <p>You must be at least 13 years old to use Kelsity.</p>
      <p>
        Come back when you're older, and we'll help you stay organized 
        with your schoolwork!
      </p>
      <a href="https://kelsity.com">Learn more about Kelsity</a>
    </div>
  );
};
```

## 3. Consent Collection Component

```typescript
// ConsentForm.tsx
interface ConsentFormProps {
  ageBracket: AgeBracket;
  onSubmit: (consents: ConsentData) => void;
}

interface ConsentData {
  termsAccepted: boolean;
  privacyAccepted: boolean;
  ferpaConsent: boolean;
  dataCollectionConsent: boolean;
  advertisingConsent?: boolean;
  marketingEmail?: boolean;
  marketingPush?: boolean;
  parentalConsent?: boolean;
  parentEmail?: string;
}

const ConsentForm: React.FC<ConsentFormProps> = ({ ageBracket, onSubmit }) => {
  const [consents, setConsents] = useState<ConsentData>({
    termsAccepted: false,
    privacyAccepted: false,
    ferpaConsent: false,
    dataCollectionConsent: false,
    advertisingConsent: false,
    marketingEmail: false,
    marketingPush: false,
    parentalConsent: false,
  });

  const isMinor = ageBracket === '13_17';

  return (
    <form onSubmit={handleSubmit} className="consent-form">
      <h2>Legal Agreements</h2>
      
      {/* Required Consents */}
      <div className="consent-section required">
        <h3>Required Agreements</h3>
        
        <label className="consent-item">
          <input
            type="checkbox"
            required
            checked={consents.termsAccepted}
            onChange={(e) => updateConsent('termsAccepted', e.target.checked)}
          />
          <span>
            I have read and agree to the{' '}
            <a href="/terms" target="_blank">Terms of Service</a> and{' '}
            <a href="/privacy" target="_blank">Privacy Policy</a>
          </span>
        </label>

        <label className="consent-item">
          <input
            type="checkbox"
            required
            checked={consents.ferpaConsent}
            onChange={(e) => updateConsent('ferpaConsent', e.target.checked)}
          />
          <span>
            I consent to Kelsity accessing and storing my educational records, 
            including course information, assignments, and grades as described 
            in the Privacy Policy
          </span>
        </label>

        <label className="consent-item">
          <input
            type="checkbox"
            required
            checked={consents.dataCollectionConsent}
            onChange={(e) => updateConsent('dataCollectionConsent', e.target.checked)}
          />
          <span>
            I understand and agree that Kelsity collects and processes my 
            personal information including profile data, academic performance, 
            and usage analytics
          </span>
        </label>

        {isMinor && (
          <>
            <label className="consent-item">
              <input
                type="checkbox"
                required
                checked={consents.parentalConsent}
                onChange={(e) => updateConsent('parentalConsent', e.target.checked)}
              />
              <span>
                I confirm that I have my parent's or guardian's permission 
                to use Kelsity
              </span>
            </label>

            <div className="parent-email">
              <label>
                Parent/Guardian Email (required):
                <input
                  type="email"
                  required
                  value={consents.parentEmail || ''}
                  onChange={(e) => updateConsent('parentEmail', e.target.value)}
                  placeholder="parent@example.com"
                />
              </label>
            </div>
          </>
        )}
      </div>

      {/* Optional Consents */}
      <div className="consent-section optional">
        <h3>Optional Preferences</h3>
        
        <label className="consent-item">
          <input
            type="checkbox"
            checked={consents.advertisingConsent}
            onChange={(e) => updateConsent('advertisingConsent', e.target.checked)}
          />
          <span>
            I agree to receive personalized content and advertisements based 
            on my interests and app usage
          </span>
        </label>

        <div className="marketing-preferences">
          <h4>Email Communications:</h4>
          <label className="consent-item">
            <input
              type="checkbox"
              checked={consents.marketingEmail}
              onChange={(e) => updateConsent('marketingEmail', e.target.checked)}
            />
            <span>
              Send me emails about new features, study tips, and special offers
            </span>
          </label>
        </div>

        <div className="marketing-preferences">
          <h4>Push Notifications:</h4>
          <label className="consent-item">
            <input
              type="checkbox"
              checked={consents.marketingPush}
              onChange={(e) => updateConsent('marketingPush', e.target.checked)}
            />
            <span>
              Send me push notifications for updates and promotions
            </span>
          </label>
        </div>

        <button 
          type="button" 
          className="select-all-optional"
          onClick={selectAllOptional}
        >
          Select All Optional
        </button>
      </div>

      <div className="consent-actions">
        <button type="submit" disabled={!isFormValid()}>
          Create Account
        </button>
      </div>
    </form>
  );
};
```

## 4. Backend API Implementation

```typescript
// signup.controller.ts
import { Request, Response } from 'express';
import { v4 as uuidv4 } from 'uuid';

interface SignupRequest {
  email: string;
  password: string;
  name: string;
  school: string;
  ageBracket: 'under_13' | '13_17' | '18_plus';
  consents: {
    termsAccepted: boolean;
    privacyAccepted: boolean;
    ferpaConsent: boolean;
    dataCollectionConsent: boolean;
    advertisingConsent?: boolean;
    marketingEmail?: boolean;
    marketingPush?: boolean;
    parentalConsent?: boolean;
    parentEmail?: string;
  };
}

export class SignupController {
  async createAccount(req: Request, res: Response) {
    const signupData: SignupRequest = req.body;
    
    // Validate age
    if (signupData.ageBracket === 'under_13') {
      return res.status(403).json({
        error: 'Users must be at least 13 years old to use Kelsity'
      });
    }
    
    // Validate required consents
    const requiredConsents = [
      'termsAccepted',
      'privacyAccepted',
      'ferpaConsent',
      'dataCollectionConsent'
    ];
    
    for (const consent of requiredConsents) {
      if (!signupData.consents[consent]) {
        return res.status(400).json({
          error: `Required consent missing: ${consent}`
        });
      }
    }
    
    // Validate parental consent for minors
    if (signupData.ageBracket === '13_17') {
      if (!signupData.consents.parentalConsent || !signupData.consents.parentEmail) {
        return res.status(400).json({
          error: 'Parental consent and email required for users under 18'
        });
      }
    }
    
    try {
      // Create user account
      const userId = await this.createUser(signupData);
      
      // Record consent
      await this.recordConsent(userId, signupData.consents, req);
      
      // Set marketing preferences
      await this.setMarketingPreferences(userId, signupData.consents);
      
      // Send verification email
      await this.sendVerificationEmail(signupData.email, signupData.name);
      
      // If minor, send parental notification
      if (signupData.ageBracket === '13_17') {
        await this.sendParentalNotification(
          signupData.consents.parentEmail!,
          signupData.name
        );
      }
      
      return res.status(201).json({
        success: true,
        userId,
        message: 'Account created successfully. Please check your email to verify.'
      });
      
    } catch (error) {
      console.error('Signup error:', error);
      return res.status(500).json({
        error: 'Failed to create account'
      });
    }
  }
  
  private async recordConsent(
    userId: string, 
    consents: any, 
    req: Request
  ): Promise<void> {
    const consentRecord = {
      id: uuidv4(),
      user_id: userId,
      consent_version: '1.0',
      terms_version: '1.0',
      privacy_version: '1.0',
      created_at: new Date(),
      ip_address: req.ip,
      user_agent: req.headers['user-agent'],
      consent_details: consents
    };
    
    await db.consentRecords.create(consentRecord);
    
    // Record individual consents
    const consentTypes = [
      { key: 'terms_of_service', granted: consents.termsAccepted },
      { key: 'privacy_policy', granted: consents.privacyAccepted },
      { key: 'ferpa_consent', granted: consents.ferpaConsent },
      { key: 'data_collection', granted: consents.dataCollectionConsent },
      { key: 'advertising', granted: consents.advertisingConsent || false },
      { key: 'parental_consent', granted: consents.parentalConsent || false }
    ];
    
    for (const consent of consentTypes) {
      await db.userConsents.create({
        id: uuidv4(),
        user_id: userId,
        consent_type_id: await this.getConsentTypeId(consent.key),
        granted: consent.granted,
        granted_at: consent.granted ? new Date() : null,
        version: '1.0'
      });
    }
  }
}
```

## 5. Consent Management Dashboard

```typescript
// ConsentDashboard.tsx
const ConsentDashboard: React.FC = () => {
  const [consents, setConsents] = useState<UserConsents>({});
  const [marketingPrefs, setMarketingPrefs] = useState<MarketingPreferences>({});
  
  return (
    <div className="consent-dashboard">
      <h2>Privacy & Consent Settings</h2>
      
      <section className="consent-history">
        <h3>Your Consent History</h3>
        <table>
          <thead>
            <tr>
              <th>Agreement</th>
              <th>Status</th>
              <th>Date</th>
              <th>Version</th>
            </tr>
          </thead>
          <tbody>
            {Object.entries(consents).map(([key, consent]) => (
              <tr key={key}>
                <td>{consent.name}</td>
                <td>{consent.granted ? 'Accepted' : 'Declined'}</td>
                <td>{formatDate(consent.grantedAt)}</td>
                <td>{consent.version}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </section>
      
      <section className="marketing-settings">
        <h3>Communication Preferences</h3>
        
        <div className="preference-group">
          <h4>Email Notifications</h4>
          <label>
            <input
              type="checkbox"
              checked={marketingPrefs.emailUpdates}
              onChange={(e) => updatePreference('emailUpdates', e.target.checked)}
            />
            Product updates and new features
          </label>
          <label>
            <input
              type="checkbox"
              checked={marketingPrefs.emailTips}
              onChange={(e) => updatePreference('emailTips', e.target.checked)}
            />
            Study tips and academic resources
          </label>
          <label>
            <input
              type="checkbox"
              checked={marketingPrefs.emailPromotions}
              onChange={(e) => updatePreference('emailPromotions', e.target.checked)}
            />
            Special offers and promotions
          </label>
        </div>
        
        <div className="preference-group">
          <h4>Push Notifications</h4>
          <label>
            <input
              type="checkbox"
              checked={marketingPrefs.pushReminders}
              onChange={(e) => updatePreference('pushReminders', e.target.checked)}
            />
            Assignment reminders
          </label>
          <label>
            <input
              type="checkbox"
              checked={marketingPrefs.pushUpdates}
              onChange={(e) => updatePreference('pushUpdates', e.target.checked)}
            />
            App updates
          </label>
        </div>
      </section>
      
      <section className="data-rights">
        <h3>Your Data Rights</h3>
        <button onClick={downloadData}>Download My Data</button>
        <button onClick={deleteAccount} className="danger">Delete My Account</button>
      </section>
    </div>
  );
};
```

## 6. Parental Consent Flow

```typescript
// ParentalConsent.tsx
const ParentalConsentFlow: React.FC = () => {
  const [token, setToken] = useState<string>('');
  const [studentInfo, setStudentInfo] = useState<StudentInfo | null>(null);
  
  useEffect(() => {
    // Extract token from URL
    const urlToken = new URLSearchParams(window.location.search).get('token');
    if (urlToken) {
      setToken(urlToken);
      fetchStudentInfo(urlToken);
    }
  }, []);
  
  return (
    <div className="parental-consent-page">
      <h1>Parental Consent for Kelsity</h1>
      
      {studentInfo && (
        <div className="student-info">
          <p>Your child <strong>{studentInfo.name}</strong> has created an 
          account on Kelsity, an academic organization app.</p>
          
          <div className="consent-details">
            <h2>What Kelsity Does</h2>
            <ul>
              <li>Helps students track assignments and due dates</li>
              <li>Provides study planning tools</li>
              <li>Enables collaboration with classmates</li>
              <li>Sends reminder notifications</li>
            </ul>
            
            <h2>Information We Collect</h2>
            <ul>
              <li>Name and email address</li>
              <li>School and grade level</li>
              <li>Course and assignment information</li>
              <li>App usage data</li>
            </ul>
            
            <h2>Your Rights as a Parent</h2>
            <ul>
              <li>Review your child's information</li>
              <li>Request changes or deletion</li>
              <li>Revoke consent at any time</li>
              <li>Contact us with concerns</li>
            </ul>
          </div>
          
          <div className="consent-actions">
            <button onClick={grantConsent} className="primary">
              I Give My Consent
            </button>
            <button onClick={denyConsent} className="secondary">
              I Do Not Give Consent
            </button>
          </div>
          
          <p className="contact-info">
            Questions? Contact us at parents@kelsity.com
          </p>
        </div>
      )}
    </div>
  );
};
```

## 7. Compliance Monitoring

```typescript
// ComplianceMonitor.ts
export class ComplianceMonitor {
  async generateComplianceReport(startDate: Date, endDate: Date) {
    const report = {
      period: { startDate, endDate },
      metrics: {
        totalSignups: 0,
        ageBreakdown: {
          under13Attempts: 0,
          age13to17: 0,
          age18Plus: 0
        },
        consentMetrics: {
          termsAcceptance: 0,
          privacyAcceptance: 0,
          ferpaConsent: 0,
          advertisingOptIn: 0,
          marketingOptIn: 0
        },
        parentalConsent: {
          requested: 0,
          granted: 0,
          denied: 0,
          pending: 0
        },
        dataRequests: {
          accessRequests: 0,
          deletionRequests: 0,
          correctionRequests: 0,
          exportRequests: 0
        }
      }
    };
    
    // Populate metrics from database
    // ... implementation
    
    return report;
  }
  
  async auditConsentRecords(userId: string) {
    const auditTrail = await db.query(`
      SELECT 
        cr.*,
        uc.consent_type_id,
        uc.granted,
        uc.granted_at,
        uc.revoked_at,
        ct.consent_key,
        ct.description
      FROM consent_records cr
      JOIN user_consents uc ON uc.user_id = cr.user_id
      JOIN consent_types ct ON ct.id = uc.consent_type_id
      WHERE cr.user_id = $1
      ORDER BY cr.created_at DESC
    `, [userId]);
    
    return auditTrail;
  }
}
```

## 8. Testing Checklist

```typescript
// SignupFlow.test.ts
describe('Signup Flow Compliance Tests', () => {
  test('should block users under 13', async () => {
    const response = await request(app)
      .post('/api/signup')
      .send({
        ageBracket: 'under_13',
        // ... other data
      });
    
    expect(response.status).toBe(403);
    expect(response.body.error).toContain('must be at least 13');
  });
  
  test('should require parental consent for 13-17', async () => {
    const response = await request(app)
      .post('/api/signup')
      .send({
        ageBracket: '13_17',
        consents: {
          parentalConsent: false
          // ... other consents
        }
      });
    
    expect(response.status).toBe(400);
    expect(response.body.error).toContain('Parental consent');
  });
  
  test('should record all consent versions', async () => {
    const userId = await createTestUser();
    const consents = await getConsentRecords(userId);
    
    expect(consents).toHaveProperty('terms_version');
    expect(consents).toHaveProperty('privacy_version');
    expect(consents).toHaveProperty('created_at');
    expect(consents).toHaveProperty('ip_address');
  });
  
  test('should allow consent withdrawal', async () => {
    const userId = await createTestUser();
    await withdrawConsent(userId, 'advertising');
    
    const consent = await getUserConsent(userId, 'advertising');
    expect(consent.granted).toBe(false);
    expect(consent.revoked_at).toBeTruthy();
  });
});
```

## 9. Deployment Checklist

- [ ] Database migrations deployed
- [ ] Consent type seeds populated
- [ ] Terms of Service v1.0 published
- [ ] Privacy Policy v1.0 published
- [ ] Age gate tested on all platforms
- [ ] Parental consent email templates ready
- [ ] Compliance reporting dashboard deployed
- [ ] Data export functionality tested
- [ ] Account deletion flow verified
- [ ] Consent audit trail logging active
- [ ] Legal review completed
- [ ] Accessibility testing passed

## 10. Monitoring and Alerts

```yaml
# monitoring-config.yaml
alerts:
  - name: "Under 13 Registration Attempts"
    condition: "age_gate.under_13_attempts > 10"
    severity: "warning"
    
  - name: "Missing Parental Consent"
    condition: "signup.minor_without_parent_consent > 0"
    severity: "critical"
    
  - name: "Consent Version Mismatch"
    condition: "consent.version != current_version"
    severity: "critical"
    
  - name: "High Consent Withdrawal Rate"
    condition: "consent.withdrawal_rate > 0.1"
    severity: "warning"
    
  - name: "GDPR Request SLA"
    condition: "gdpr_request.pending_days > 25"
    severity: "critical"
```