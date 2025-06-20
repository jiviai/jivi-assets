# Samsung Health Data - MongoDB Schema Documentation

## 📊 Overview

This document outlines the MongoDB collection structure for Samsung Health data synchronization in a Python-based application. The data encompasses various health metrics including activity tracking, vital signs, body composition, and sleep patterns. All collections use a consistent approach with `user_id` as the primary identifier and standardized timestamps.

## 🗄️ Database Architecture

### Database Name: `samsung_health_db`

The database consists of 7 primary collections, each storing specific types of health metrics:

1. **activities**
2. **blood_glucose**
3. **blood_pressure**
4. **body_composition**
5. **heart_rate**
6. **sleep_tracking**
7. **user_profiles**

---

## 📋 Collection Schemas

### 1. Activities Collection (`activities`)

Stores daily physical activity metrics including steps, calories, and active minutes.

```javascript
{
  "_id": ObjectId,
  "user_id": String,
  "date": Date,                    // YYYY-MM-DD format
  "timestamp": DateTime,           // YYYY-MM-DD HH:MM:SS
  "daily_step_count": Number,
  "daily_step_goal": Number,
  "step_goal_achieved": Boolean,   // Calculated field
  "daily_active_calories": Number,
  "daily_active_minutes": Number,
  "created_at": DateTime,
  "updated_at": DateTime
}
```

**Indexes:**
- Unique Compound: `{ user_id: 1, timestamp: 1 }` - Prevents duplicates
- Single: `{ date: -1 }`

---

### 2. Blood Glucose Collection (`blood_glucose`)

Tracks blood glucose readings with HbA1c values.

```javascript
{
  "_id": ObjectId,
  "user_id": String,
  "reading_id": String,            // UUID from source
  "timestamp": DateTime,           // YYYY-MM-DD HH:MM:SS
  "bg_value": Number,              // in mmol/L
  "bg_type": Number,               // -1: Unknown, 90001: Whole Blood, 90002: Plasma, 90003: Serum
  "bg_type_label": String,         // Human-readable type
  "hba1c_value": Number,           // in percentage
  "unit": String,                  // "mmol/L"
  "created_at": DateTime
}
```

**Indexes:**
- Unique Compound: `{ user_id: 1, timestamp: 1 }` - Prevents duplicates
- Single: `{ reading_id: 1 }`

---

### 3. Blood Pressure Collection (`blood_pressure`)

Records blood pressure measurements with pulse rate.

```javascript
{
  "_id": ObjectId,
  "user_id": String,
  "timestamp": DateTime,           // YYYY-MM-DD HH:MM:SS
  "systolic": Number,              // mmHg
  "diastolic": Number,             // mmHg
  "pulse_rate": Number,            // bpm
  "bp_category": String,           // "Normal", "Elevated", "High", etc.
  "unit": String,                  // "mmHg"
  "created_at": DateTime
}
```

**BP Categories Logic:**
- Normal: Systolic < 120 AND Diastolic < 80
- Elevated: Systolic 120-129 AND Diastolic < 80
- High Stage 1: Systolic 130-139 OR Diastolic 80-89
- High Stage 2: Systolic ≥ 140 OR Diastolic ≥ 90

**Indexes:**
- Unique Compound: `{ user_id: 1, timestamp: 1 }` - Prevents duplicates
- Single: `{ bp_category: 1 }`

---

### 4. Body Composition Collection (`body_composition`)

Comprehensive body metrics including weight, BMR, and composition percentages.

```javascript
{
  "_id": ObjectId,
  "user_id": String,
  "timestamp": DateTime,           // YYYY-MM-DD HH:MM:SS
  "weight": Number,                // in pounds (as per user preference)
  "weight_kg": Number,             // Converted to kg for calculations
  "bmr": Number,                   // Basal Metabolic Rate (calories)
  "body_fat_percentage": Number,   // %
  "body_water_percentage": Number, // %
  "fat_mass": Number,              // kg
  "skeletal_muscle_mass": Number,  // kg
  "bmi": Number,                   // Calculated field
  "bmi_category": String,          // "Underweight", "Normal", "Overweight", "Obese"
  "created_at": DateTime
}
```

**BMI Calculation:** `weight_kg / (height_m)²`

**Indexes:**
- Unique Compound: `{ user_id: 1, timestamp: 1 }` - Prevents duplicates
- Single: `{ bmi: 1 }`

---

### 5. Heart Rate Collection (`heart_rate`)

Daily heart rate statistics.

```javascript
{
  "_id": ObjectId,
  "user_id": String,
  "date": Date,                    // YYYY-MM-DD
  "average_heart_rate": Number,    // bpm
  "minimum_heart_rate": Number,    // bpm
  "maximum_heart_rate": Number,    // bpm
  "resting_heart_rate": Number,    // Calculated/inferred
  "created_at": DateTime,
  "updated_at": DateTime
}
```

**Indexes:**
- Unique Compound: `{ user_id: 1, date: 1 }` - Prevents duplicates

---

### 6. Sleep Tracking Collection (`sleep_tracking`)

Detailed sleep session data with quality metrics.

```javascript
{
  "_id": ObjectId,
  "user_id": String,
  "sleep_id": String,              // UUID from source
  "sleep_date": Date,              // YYYY-MM-DD
  "sleep_start": DateTime,         // YYYY-MM-DD HH:MM:SS
  "sleep_end": DateTime,           // YYYY-MM-DD HH:MM:SS
  "duration_hours": Number,        // Calculated field
  "sleep_quality": Number,         // 0-100 scale
  "sleep_score": Number,           // 0-100 scale
  "average_heart_rate": Number,    // bpm during sleep
  "average_respiratory_rate": Number, // breaths per minute
  "skin_temp_min": Number,         // °C
  "skin_temp_max": Number,         // °C
  "total_snoring_time": Number,    // minutes
  "created_at": DateTime
}
```

**Indexes:**
- Unique Compound: `{ user_id: 1, sleep_id: 1 }` - Prevents duplicate sessions
- Compound: `{ user_id: 1, sleep_date: -1 }`

---

### 7. User Profiles Collection (`user_profiles`)

User demographic and preference data.

```javascript
{
  "_id": ObjectId,
  "user_id": String,
  "age": Number,
  "gender": String,                // "M" or "F"
  "height": Number,                // cm
  "height_m": Number,              // Converted to meters
  "units": {
    "height": String,              // "cm"
    "weight": String,              // "lb"
    "blood_glucose": String,       // "mmol/L"
    "blood_pressure": String,      // "mmHg"
    "body_fat": String,            // "%"
    "hba1c": String               // "%"
  },
  "created_at": DateTime,
  "updated_at": DateTime
}
```

**Indexes:**
- Unique: `{ user_id: 1 }` - One profile per user

---

## 🐍 Python Implementation Guidelines

### MongoDB Connection Setup
```python
from pymongo import MongoClient
from datetime import datetime

# MongoDB connection
client = MongoClient('mongodb://localhost:27017/')
db = client['samsung_health_db']
```

### Timestamp Conversion
```python
from datetime import datetime

def convert_epoch_to_datetime(epoch_ms):
    """Convert epoch milliseconds to datetime object"""
    return datetime.fromtimestamp(epoch_ms / 1000)

def format_datetime(dt):
    """Format datetime to YYYY-MM-DD HH:MM:SS string"""
    return dt.strftime('%Y-%m-%d %H:%M:%S')
```

### Upsert Operations
```python
def upsert_activity(collection, user_id, data):
    """Upsert activity data with null check"""
    if data.get('dailyStepCount') is None:
        return None
    
    timestamp = convert_epoch_to_datetime(data['timeStamp'])
    
    document = {
        'user_id': user_id,
        'timestamp': timestamp,
        'date': timestamp.date(),
        'daily_step_count': data['dailyStepCount'],
        'daily_step_goal': data['dailyStepGoal'],
        'step_goal_achieved': data['dailyStepCount'] >= data['dailyStepGoal'],
        'daily_active_calories': data['dailyActiveCaloriesBurnt'],
        'daily_active_minutes': data['dailyActiveMinutes'],
        'updated_at': datetime.now()
    }
    
    # Remove None values
    document = {k: v for k, v in document.items() if v is not None}
    
    # Upsert based on user_id and timestamp
    result = collection.update_one(
        {'user_id': user_id, 'timestamp': timestamp},
        {'$set': document, '$setOnInsert': {'created_at': datetime.now()}},
        upsert=True
    )
    
    return result
```

### Index Creation
```python
def create_indexes():
    """Create all required indexes for collections"""
    
    # Activities indexes
    db.activities.create_index([('user_id', 1), ('timestamp', 1)], unique=True)
    db.activities.create_index([('date', -1)])
    
    # Blood glucose indexes
    db.blood_glucose.create_index([('user_id', 1), ('timestamp', 1)], unique=True)
    db.blood_glucose.create_index('reading_id')
    
    # Blood pressure indexes
    db.blood_pressure.create_index([('user_id', 1), ('timestamp', 1)], unique=True)
    db.blood_pressure.create_index('bp_category')
    
    # Body composition indexes
    db.body_composition.create_index([('user_id', 1), ('timestamp', 1)], unique=True)
    db.body_composition.create_index('bmi')
    
    # Heart rate indexes
    db.heart_rate.create_index([('user_id', 1), ('date', 1)], unique=True)
    
    # Sleep tracking indexes
    db.sleep_tracking.create_index([('user_id', 1), ('sleep_id', 1)], unique=True)
    db.sleep_tracking.create_index([('user_id', 1), ('sleep_date', -1)])
    
    # User profiles index
    db.user_profiles.create_index('user_id', unique=True)
```

---

## 📊 Common Aggregation Patterns

### Python Aggregation Examples

```python
# Weekly step average
def get_weekly_step_average(user_id, start_date, end_date):
    pipeline = [
        {
            '$match': {
                'user_id': user_id,
                'date': {'$gte': start_date, '$lte': end_date}
            }
        },
        {
            '$group': {
                '_id': {'$week': '$date'},
                'avg_steps': {'$avg': '$daily_step_count'},
                'total_calories': {'$sum': '$daily_active_calories'}
            }
        }
    ]
    return list(db.activities.aggregate(pipeline))

# Body composition trends
def get_body_composition_trends(user_id):
    pipeline = [
        {'$match': {'user_id': user_id}},
        {'$sort': {'timestamp': 1}},
        {
            '$group': {
                '_id': {
                    'year': {'$year': '$timestamp'},
                    'month': {'$month': '$timestamp'}
                },
                'avg_weight': {'$avg': '$weight_kg'},
                'avg_body_fat': {'$avg': '$body_fat_percentage'}
            }
        }
    ]
    return list(db.body_composition.aggregate(pipeline))
```

---

## ✅ Technical Implementation Checklist

### Database Setup
- [ ] Install MongoDB and PyMongo
- [ ] Create `samsung_health_db` database
- [ ] Create all 7 collections
- [ ] Run index creation script for all collections

### Data Processing Pipeline
- [ ] Implement epoch to datetime conversion functions
- [ ] Create upsert functions for each collection type
- [ ] Implement null check logic for each data type
- [ ] Create batch processing logic for S3 data files

### Collection-Specific Implementation

#### Activities
- [ ] Parse activity_data_list from JSON
- [ ] Convert timestamps to datetime
- [ ] Calculate step_goal_achieved field
- [ ] Implement upsert with user_id + timestamp

#### Blood Glucose
- [ ] Parse blood_glucose_reading array
- [ ] Map bg_type codes to labels
- [ ] Handle null HbA1c values
- [ ] Implement upsert with user_id + timestamp

#### Blood Pressure
- [ ] Parse blood_pressure_daily_data_list
- [ ] Calculate BP category based on values
- [ ] Handle zero pulse rate values
- [ ] Implement upsert with user_id + timestamp

#### Body Composition
- [ ] Parse body_comp_data_item_list
- [ ] Convert weight from pounds to kg
- [ ] Calculate BMI and BMI category
- [ ] Implement upsert with user_id + timestamp

#### Heart Rate
- [ ] Parse heart_rate_list
- [ ] Convert date string to Date object
- [ ] Handle missing resting heart rate
- [ ] Implement upsert with user_id + date

#### Sleep Tracking
- [ ] Parse sleep_duration_data_list and nested sleep_sample_data_list
- [ ] Calculate sleep duration in hours
- [ ] Handle zero values for optional metrics
- [ ] Implement upsert with user_id + sleep_id

#### User Profiles
- [ ] Parse user_info object
- [ ] Convert height to meters
- [ ] Map unit preferences
- [ ] Implement upsert with user_id only

### Testing & Validation
- [ ] Test upsert operations for new records
- [ ] Test upsert operations for existing records with changes
- [ ] Verify unique indexes prevent duplicates
- [ ] Test null value handling
- [ ] Verify timestamp conversions are correct

### Production Deployment
- [ ] Set up S3 file monitoring/polling
- [ ] Implement error logging and retry mechanism
- [ ] Create monitoring dashboard for data ingestion
- [ ] Set up automated backups
- [ ] Document API endpoints for data access

### Post-Deployment
- [ ] Monitor index performance
- [ ] Track upsert success/failure rates
- [ ] Set up alerts for data anomalies
- [ ] Create data quality reports
- [ ] Schedule regular index maintenance

---

## 📝 Notes

- All timestamps are stored in UTC format
- The system uses upsert operations to handle both inserts and updates
- Duplicate prevention is enforced at the database level through unique compound indexes
- Only null checks are performed; no other data validation is implemented
- The `updated_at` field tracks when records were last modified

---

*Last Updated: June 2025*
*Version: 2.0*

Rishabh A. (rishabh@jivi.ai)