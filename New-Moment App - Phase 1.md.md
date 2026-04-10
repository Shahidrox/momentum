# 🧱 ✅ Final Redesigned Models

```python
from sqlalchemy import Column, Integer, ForeignKey, DateTime, Float
from sqlalchemy.orm import relationship
from datetime import datetime

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)

    # 🔗 Relationships
    workouts = relationship("Workout", back_populates="user")
    sessions = relationship("WorkoutSession", back_populates="user")
    daily_stats = relationship("DailyStats", back_populates="user")

class Workout(Base):
    __tablename__ = "workouts"

    id = Column(Integer, primary_key=True, index=True)

    user_id = Column(Integer, ForeignKey("users.id"), index=True)
    program_exercise_id = Column(Integer, ForeignKey("program_exercises.id"), index=True)
    session_id = Column(Integer, ForeignKey("workout_sessions.id"), index=True)

    duration = Column(Integer)  # seconds
    reps = Column(Integer, nullable=True)

    # 🔥 Core scoring
    intensity = Column(Integer, default=3)  # 1–5 scale
    score = Column(Float, default=0)

    # 🧠 Sensor summary (NEW)
    avg_intensity = Column(Float, nullable=True)
    avg_stability = Column(Float, nullable=True)
    total_steps = Column(Integer, nullable=True)

    created_at = Column(DateTime, default=datetime.utcnow, index=True)

    # 🔗 Relationships
    user = relationship("User", back_populates="workouts")
    program_exercise = relationship("ProgramExercise")
    session = relationship("WorkoutSession", back_populates="workouts")


class WorkoutSession(Base):
    __tablename__ = "workout_sessions"

    id = Column(Integer, primary_key=True, index=True)

    user_id = Column(Integer, ForeignKey("users.id"), index=True)
    program_id = Column(Integer, ForeignKey("programs.id"), nullable=True)

    total_duration = Column(Integer, default=0)
    total_score = Column(Float, default=0)

    total_exercises = Column(Integer, default=0)  # 🔥 NEW

    created_at = Column(DateTime, default=datetime.utcnow, index=True)

    # 🔗 Relationships
    user = relationship("User", back_populates="sessions")
    workouts = relationship("Workout", back_populates="session", cascade="all, delete")

class DailyStats(Base):
    __tablename__ = "daily_stats"

    id = Column(Integer, primary_key=True, index=True)

    user_id = Column(Integer, ForeignKey("users.id"), index=True)
    date = Column(Date, index=True)

    total_workouts = Column(Integer, default=0)
    total_duration = Column(Integer, default=0)
    total_score = Column(Float, default=0)

    avg_intensity = Column(Float, default=0)   # 🔥 NEW
    avg_stability = Column(Float, default=0)   # 🔥 NEW

    total_steps = Column(Integer, default=0)   # 🔥 NEW

    streak_day = Column(Integer, default=0)

    # 🔒 Prevent duplicate rows per day
    __table_args__ = (
        UniqueConstraint("user_id", "date", name="unique_user_date"),
    )

    # 🔗 Relationships
    user = relationship("User", back_populates="daily_stats")

class WorkoutSensorData(Base):
    __tablename__ = "workout_sensor_data"

    id = Column(Integer, primary_key=True)
    workout_id = Column(Integer, ForeignKey("workouts.id"))

    accel_x = Column(Float)
    accel_y = Column(Float)
    accel_z = Column(Float)

    gyro_x = Column(Float)
    gyro_y = Column(Float)
    gyro_z = Column(Float)

    timestamp = Column(DateTime)
```

### 🔥 Final Architecture (You Now Have)

Workout → raw exercise<br/>
WorkoutSession → grouped session<br/>
DailyStats → dashboard analytics<br/>


