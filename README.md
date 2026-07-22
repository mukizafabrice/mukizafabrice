# Fabrice Nshimyumukiza

**Senior Full-Stack Developer | Systems Architect | Kigali, Rwanda**

```
┌─────────────────────────────────────────────────────────────┐
│  Building scalable enterprise systems with clean code       │
│  5+ years | Healthcare Systems | System Architecture        │
└─────────────────────────────────────────────────────────────┘
```

---

## 🏗️ Architecture & Tech Stack

```
┌──────────────────────────────────────────────────────────┐
│                    SYSTEM ARCHITECTURE                    │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  ┌─────────────┐      ┌─────────────┐     ┌──────────┐  │
│  │  Frontend   │──────│   Backend   │────▶│ Database │  │
│  │   React     │      │  Node.js    │     │ PostgeSQL│  │
│  │  Tailwind   │      │  Frappe     │     │ MongoDB  │  │
│  └─────────────┘      └─────────────┘     └──────────┘  │
│        ▲                     ▲                              │
│        │                     │                              │
│  ┌─────┴─────┐      ┌────────┴────────┐                   │
│  │  Mobile   │      │  Cache Layer    │                   │
│  │React Native   │      │  Redis      │                   │
│  └───────────┘      └─────────────────┘                   │
│                                                            │
└──────────────────────────────────────────────────────────┘
```

**Languages:** JavaScript · TypeScript · Python · PHP · Java  
**Frontend:** React · React Native · Tailwind CSS  
**Backend:** Node.js · Frappe/ERPNext · Express  
**Database:** PostgreSQL · MongoDB · Redis  
**DevOps:** Docker · CI/CD · AWS · Linux  

---

## 🔧 Key Projects & Code Examples

### 1️⃣ Rwanda MOH EMR System (Frappe/ERPNext)

**Architecture Overview:**
```
┌────────────────────────────────────────────────────────┐
│           Electronic Medical Records System             │
├────────────────────────────────────────────────────────┤
│                                                         │
│  Patient Module        Clinic Module                   │
│  ├─ Patient Registry   ├─ Appointments                 │
│  ├─ Medical History    ├─ Billing                      │
│  └─ Lab Results        └─ Inventory                    │
│                                                         │
│  ✓ Multi-tenant architecture (15+ facilities)          │
│  ✓ Role-based access control (Doctor/Nurse/Admin)     │
│  ✓ HIPAA-adjacent compliance                           │
│  ✓ Real-time reporting                                 │
│                                                         │
└────────────────────────────────────────────────────────┘
```

**Performance Metrics:**
```python
# Database Query Optimization
Before: SELECT * FROM patient_records WHERE clinic_id = X  # 5.2s
After:  SELECT id, name, age FROM patient_records 
        WHERE clinic_id = X 
        AND created_date > DATE_SUB(NOW(), INTERVAL 90 DAY)  # 0.3s

Result: ⚡ 40% performance improvement
```

**Implementation (Frappe Custom Doctype):**
```python
# hooks.py - Custom Patient Hooks
from frappe.utils import now_datetime

def validate_patient(doc, method):
    """Validate patient data and HIPAA compliance"""
    if doc.date_of_birth:
        doc.age = calculate_age(doc.date_of_birth)
    
    # Audit logging
    log_access(
        user=frappe.session.user,
        action="view_patient",
        patient_id=doc.name,
        timestamp=now_datetime()
    )

# Custom Report: Patient Summary
SELECT 
    p.name as patient_id,
    p.patient_name,
    COUNT(a.name) as total_appointments,
    MAX(a.appointment_date) as last_visit,
    SUM(b.amount) as total_charges
FROM `tabPatient` p
LEFT JOIN `tabAppointment` a ON p.name = a.patient
LEFT JOIN `tabBilling` b ON p.name = b.patient
WHERE p.clinic = %(clinic)s
GROUP BY p.name
ORDER BY last_visit DESC
```

---

### 2️⃣ Tira Car Rental Platform (React + Node.js)

**System Design:**
```
┌─────────────────────────────────────────────────────────┐
│           Tira - Multi-Portal Fleet Management          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ADMIN PORTAL          DRIVER PORTAL      CUSTOMER APP  │
│  ├─ Fleet Mgmt         ├─ Schedule         ├─ Browse    │
│  ├─ Bookings           ├─ Trips            ├─ Book      │
│  ├─ Reports            └─ Earnings         ├─ Track     │
│  └─ Analytics                              └─ Rate      │
│                                                          │
│         ◀─────── Shared API (Node.js) ────────▶          │
│                                                          │
│  Database: MongoDB    Cache: Redis    Queue: Bull      │
│  Status: 99.2% uptime | 100+ bookings/day              │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Backend API Structure:**
```javascript
// Express API Routes - Car Rental System
const express = require('express');
const router = express.Router();
const carController = require('../controllers/carController');
const { auth, authorize } = require('../middleware/auth');

// Public Routes
router.get('/cars/available', carController.getAvailableCars);
router.get('/cars/:id', carController.getCarDetails);

// Authenticated Routes
router.use(auth);

// Customer Routes
router.post('/bookings', authorize('customer'), carController.createBooking);
router.get('/bookings/my', carController.getMyBookings);
router.post('/bookings/:id/cancel', carController.cancelBooking);

// Driver Routes
router.get('/trips/assigned', authorize('driver'), carController.getAssignedTrips);
router.patch('/trips/:id/status', authorize('driver'), carController.updateTripStatus);

// Admin Routes
router.get('/fleet/analytics', authorize('admin'), carController.getFleetAnalytics);
router.post('/cars', authorize('admin'), carController.createCar);
router.patch('/cars/:id', authorize('admin'), carController.updateCar);

module.exports = router;
```

**Database Schema (MongoDB):**
```javascript
// Booking Schema with Timestamps & Status Tracking
db.createCollection('bookings', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['customer_id', 'car_id', 'pickup_date', 'status'],
      properties: {
        _id: { bsonType: 'objectId' },
        customer_id: { bsonType: 'objectId', description: 'Customer reference' },
        car_id: { bsonType: 'objectId', description: 'Vehicle reference' },
        pickup_date: { bsonType: 'date' },
        return_date: { bsonType: 'date' },
        status: { enum: ['pending', 'confirmed', 'ongoing', 'completed', 'cancelled'] },
        total_cost: { bsonType: 'decimal128' },
        payment_status: { enum: ['unpaid', 'partial', 'paid'] },
        created_at: { bsonType: 'date' },
        updated_at: { bsonType: 'date' }
      }
    }
  }
});

// Indexes for Performance
db.bookings.createIndex({ customer_id: 1, created_at: -1 });
db.bookings.createIndex({ car_id: 1, status: 1 });
db.bookings.createIndex({ pickup_date: 1, return_date: 1 });
```

**React Component Example:**
```jsx
// BookingForm.jsx - Multi-step booking with validation
import React, { useState, useEffect } from 'react';
import { validateDates, calculatePrice } from '../utils/bookingUtils';

export const BookingForm = ({ carId }) => {
  const [booking, setBooking] = useState({
    pickupDate: null,
    returnDate: null,
    carId: carId,
    status: 'pending'
  });
  const [errors, setErrors] = useState({});
  const [loading, setLoading] = useState(false);

  const handleDateChange = (field, value) => {
    setBooking(prev => ({ ...prev, [field]: value }));
    validateDates(booking.pickupDate, booking.returnDate);
  };

  const submitBooking = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/bookings', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(booking)
      });
      
      if (!response.ok) throw new Error('Booking failed');
      
      const data = await response.json();
      return data;
    } catch (error) {
      setErrors({ submit: error.message });
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={(e) => { e.preventDefault(); submitBooking(); }}>
      <input type="date" onChange={(e) => handleDateChange('pickupDate', e.target.value)} />
      <input type="date" onChange={(e) => handleDateChange('returnDate', e.target.value)} />
      <button disabled={loading} type="submit">Book Now</button>
    </form>
  );
};
```

---

### 3️⃣ UBridge - Cooperative Governance System

**Data Model:**
```
Cooperative
├─ Members
│  ├─ role (Admin, Supervisor, Member)
│  ├─ status (active, inactive, suspended)
│  └─ joined_date
├─ Votes
│  ├─ vote_id
│  ├─ proposal_id
│  ├─ member_id
│  └─ choice (yes, no, abstain)
└─ Transactions
   ├─ transaction_id
   ├─ member_id
   ├─ amount
   └─ timestamp
```

**Vote Processing Algorithm:**
```python
# Smart voting system with real-time results
class VotingSystem:
    def __init__(self, proposal_id):
        self.proposal = Proposal.get(proposal_id)
        self.votes = Vote.get_votes(proposal_id)
    
    def calculate_results(self):
        """Calculate voting results in real-time"""
        total_votes = len(self.votes)
        yes_votes = self.votes.filter(choice='yes').count()
        no_votes = self.votes.filter(choice='no').count()
        
        approval_rate = (yes_votes / total_votes) * 100 if total_votes > 0 else 0
        
        return {
            'yes': yes_votes,
            'no': no_votes,
            'total': total_votes,
            'approval_percentage': round(approval_rate, 2),
            'passed': approval_rate >= self.proposal.required_percentage
        }
    
    def notify_members(self, result):
        """Async notification to all members"""
        for member in Member.get_active():
            send_notification(
                member_id=member.id,
                title=f"Vote Result: {self.proposal.title}",
                message=f"Result: {'PASSED' if result['passed'] else 'REJECTED'}"
            )

# Usage
voting = VotingSystem(proposal_id='PROP-001')
results = voting.calculate_results()
voting.notify_members(results)
```

---

## 📈 Performance Optimizations

### Database Query Optimization
```sql
-- ❌ BEFORE: Full table scan (2.5s)
SELECT * FROM appointments WHERE clinic_id = 5 AND status = 'completed';

-- ✅ AFTER: Indexed query (0.08s)
SELECT id, patient_id, appointment_date, status 
FROM appointments 
WHERE clinic_id = 5 
  AND status = 'completed'
  AND appointment_date > DATE_SUB(NOW(), INTERVAL 30 DAY);

CREATE INDEX idx_clinic_status_date ON appointments(clinic_id, status, appointment_date);

-- Result: 31x performance improvement ⚡
```

### API Response Caching
```javascript
// Redis caching layer for healthcare data
const redis = require('redis');
const client = redis.createClient();

async function getPatientRecords(clinicId) {
  const cacheKey = `patients:clinic:${clinicId}`;
  
  // Check cache first
  let cached = await client.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // Cache miss - fetch from DB
  const patients = await db.Patient.find({ clinic_id: clinicId });
  
  // Store in cache for 1 hour
  await client.setex(cacheKey, 3600, JSON.stringify(patients));
  
  return patients;
}
```

### Frontend Performance
```javascript
// Code splitting with lazy loading
const PatientModule = React.lazy(() => import('./modules/Patient'));
const BillingModule = React.lazy(() => import('./modules/Billing'));

export const App = () => (
  <Suspense fallback={<LoadingSpinner />}>
    <Routes>
      <Route path="/patients" element={<PatientModule />} />
      <Route path="/billing" element={<BillingModule />} />
    </Routes>
  </Suspense>
);
```

---

## 🔐 Security Implementations

```python
# HIPAA-Adjacent Compliance Layer
from flask import request
from functools import wraps
import logging

class AuditLogger:
    @staticmethod
    def log_access(user_id, resource_type, resource_id, action):
        """Log all access to sensitive patient data"""
        logging.info(f"""
        AUDIT_LOG: 
        User: {user_id}
        Resource: {resource_type}
        ID: {resource_id}
        Action: {action}
        Timestamp: {datetime.now()}
        IP: {request.remote_addr}
        """)

def require_permission(required_role):
    """Role-based access control decorator"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            user_role = get_user_role(current_user)
            if user_role not in required_role:
                AuditLogger.log_access(
                    current_user.id, 
                    'unauthorized_access',
                    kwargs.get('resource_id'),
                    'DENIED'
                )
                return {'error': 'Access Denied'}, 403
            return func(*args, **kwargs)
        return wrapper
    return decorator

@require_permission(['doctor', 'nurse', 'admin'])
def view_patient_records(patient_id):
    """Only authorized roles can view sensitive patient data"""
    AuditLogger.log_access(current_user.id, 'patient', patient_id, 'VIEW')
    return get_patient_data(patient_id)
```

---

## 📊 Metrics & Impact

```
Healthcare System Optimization
├─ Database Performance
│  └─ Query speed improved: 5.2s → 0.3s (40% faster) ⚡
├─ Multi-tenant Architecture
│  └─ Facilities supported: 15+ clinics & growing 🏥
├─ User Experience
│  └─ Manual data entry reduced: 85% automation ✓
├─ System Reliability
│  └─ Uptime: 99.2% (Tira platform) 🟢
└─ Team Leadership
   └─ Junior developers mentored: 3+ ✓

Freelance Impact
├─ Projects completed: 12+
├─ Client satisfaction: 98% retention
├─ Average project value: $2K-$8K USD
└─ Sectors: Healthcare · Finance · E-commerce
```

---

## 🚀 Current Learning & Growth

```
Education
├─ University of the People
│  └─ Computer Science (In Progress)
├─ RP Ngoma College
│  └─ Information Technology (In Progress)
└─ Self-Directed
   ├─ AI/ML in Healthcare Systems
   ├─ Distributed Systems Architecture
   └─ Advanced Database Optimization

Tech Roadmap
├─ Kubernetes Orchestration
├─ GraphQL API Design
├─ Event-Driven Architecture (Kafka)
└─ Machine Learning in Healthcare
```

---

## 📞 Get in Touch

**[LinkedIn](https://linkedin.com/in/nshimyumukiza-fabrice-b55751256)** | 
**[GitHub](https://github.com/mukizafabrice)** | 
**[Portfolio](https://fabrice250.netlify.app)** | 
**[Email](mailto:mukizafabrice18@gmail.com)**

---

## 🔗 Project Repositories

| Project | Status | Tech Stack | Link |
|---------|--------|-----------|------|
| Rwanda MOH EMR | 🔴 Active Dev | Frappe/ERPNext · Python | [View](https://github.com/mukizafabrice/moh-emr) |
| Tira Car Rental | 🟢 Live | React · Node.js · MongoDB | [View](https://github.com/mukizafabrice/tira-car-rental) |
| UBridge Cooperative | 🟢 Active | React · Node.js | [View](https://github.com/mukizafabrice/ubridge) |
| Max Cure Clinic | 🟢 Production | Frappe/ERPNext | [View](https://github.com/mukizafabrice) |

---

```
╔════════════════════════════════════════════════════════════╗
║  "Building systems, not just applications"                ║
║  Senior Full-Stack Developer | Systems Architect          ║
║  Kigali, Rwanda | 5+ Years Experience                     ║
╚════════════════════════════════════════════════════════════╝
```
