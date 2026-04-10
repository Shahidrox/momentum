Moment App - Phase 1

This document defines the **data model, scoring system, and aggregation logic** for the Naboso Moment App.

The goal is to enable:

- Workout tracking
- Moment score calculation
- Daily progress analytics
- 7-day visualization
- Future gamification (streaks, leaderboard)

**🚀 Enhancements for Phase 1**

To support **moment score and progress tracking**, we introduce:

- Update Workout
- Add WorkoutSession
- Add DailyStats

**🏋️ Workout Table (Updated)**

class Workout(Base):  
\_\_tablename\_\_= "workouts"  
<br/>id = Column(Integer, primary_key=True)  
<br/>user_id = Column(Integer, ForeignKey("users.id"))  
program_exercise_id = Column(Integer, ForeignKey("program_exercises.id"))  
<br/>duration = Column(Integer) # seconds  
reps = Column(Integer, nullable=True)  
<br/>\# New fields  
intensity = Column(Integer, default=3) # 1-5 scale  
score = Column(Float, default=0)  
<br/>session_id = Column(Integer, ForeignKey("workout_sessions.id"))  
<br/>created_at = Column(DateTime, default=datetime.utcnow)

# 📦 WorkoutSession Table (New)

Groups multiple workouts into one session.

class WorkoutSession(Base):  
\_\_tablename\_\_= "workout_sessions"  
<br/>id = Column(Integer, primary_key=True)  
<br/>user_id = Column(Integer, ForeignKey("users.id"))  
program_id = Column(Integer, ForeignKey("programs.id"), nullable=True)  
<br/>total_duration = Column(Integer, default=0)  
total_score = Column(Float, default=0)  
<br/>created_at = Column(DateTime, default=datetime.utcnow)

# 📊 DailyStats Table (New)

Used for:

- Dashboard
- Progress page
- 7-day graph
- Streak tracking

class DailyStats(Base):  
\_\_tablename\_\_= "daily_stats"  
<br/>id = Column(Integer, primary_key=True)  
<br/>user_id = Column(Integer, ForeignKey("users.id"))  
date = Column(Date)  
<br/>total_workouts = Column(Integer, default=0)  
total_duration = Column(Integer, default=0)  
total_score = Column(Float, default=0)  
<br/>streak_day = Column(Integer, default=0)

# ⚙️ Score Calculation Logic

Basic scoring formula:

def calculate_score(duration, intensity, reps=0):  
base = duration / 60 \* intensity # convert seconds → minutes  
rep_bonus = reps \* 0.1 if reps else 0  
return round(base + rep_bonus, 2)

# 🔄 Workout Logging Flow

When a user completes an exercise:

### Step 1: Calculate score

### Step 2: Save Workout

### Step 3: Update DailyStats

## Example Implementation

def log_workout(db, user_id, data):  
score = calculate_score(  
data.duration,  
data.intensity,  
data.reps  
)  
<br/>workout = Workout(  
user_id=user_id,  
program_exercise_id=data.program_exercise_id,  
duration=data.duration,  
reps=data.reps,  
intensity=data.intensity,  
score=score,  
session_id=data.session_id  
)  
<br/>db.add(workout)  
db.commit()  
db.refresh(workout)  
<br/>update_daily_stats(db, user_id, workout)  
<br/>return workout

# 📅 DailyStats Update Logic

from datetime import date  
<br/>def update_daily_stats(db, user_id, workout):  
today = date.today()  
<br/>stats = db.query(DailyStats).filter_by(  
user_id=user_id,  
date=today  
).first()  
<br/>if not stats:  
stats = DailyStats(  
user_id=user_id,  
date=today,  
total_workouts=0,  
total_duration=0,  
total_score=0,  
streak_day=1  
)  
db.add(stats)  
<br/>stats.total_workouts += 1  
stats.total_duration += workout.duration  
stats.total_score += workout.score  
<br/>db.commit()

# 📈 Progress Features

## 1\. 7-Day Stats API

def get*7_day_stats(db, user_id):  
from datetime import timedelta, date  
<br/>today = date.today()  
last_7_days = \[today - timedelta(days=i) for i in range(7)\]  
<br/>data = db.query(DailyStats).filter(  
DailyStats.user_id == user_id,  
DailyStats.date.in*(last_7_days)  
).all()  
<br/>return data

## 2\. Moment Score

### Simple version

moment_score = sum(last_7_days.total_score)

### Advanced version

moment_score = (consistency \* 0.4) + (volume \* 0.3) + (intensity \* 0.3)

# 🔥 Key Benefits

This architecture provides:

- ✅ Scalable workout tracking
- ✅ Clean score system
- ✅ Fast dashboard queries
- ✅ Easy leaderboard integration (Phase 2)
- ✅ Supports gamification (streaks, challenges)
