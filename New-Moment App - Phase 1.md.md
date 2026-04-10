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


### Core Logic

```python
from datetime import date

def process_workout(db, user_id, data):
    
    # 🎯 1. Calculate score
    score = calculate_score(
        data.duration,
        data.intensity,
        data.avg_stability
    )

    # 🏋️ 2. Create Workout
    workout = Workout(
        user_id=user_id,
        program_exercise_id=data.program_exercise_id,
        session_id=data.session_id,
        duration=data.duration,
        reps=data.reps,
        intensity=data.intensity,
        score=score,
        avg_intensity=data.avg_intensity,
        avg_stability=data.avg_stability,
        total_steps=data.total_steps,
    )

    db.add(workout)
    db.commit()
    db.refresh(workout)

    # 📦 3. Update Session
    update_session(db, data.session_id, data, score)

    # 📊 4. Update Daily Stats
    update_daily_stats(db, user_id, data, score)

    return workout

# Score Calculation
def calculate_score(duration, intensity, stability):
    base = (duration / 60) * intensity
    stability_bonus = (stability or 0) * 0.5
    return round(base + stability_bonus, 2)

# Create/Update WorkoutSession
def update_session(db, session_id, data, score):
    session = db.query(WorkoutSession).filter_by(id=session_id).first()

    if not session:
        return

    session.total_duration += data.duration
    session.total_score += score
    session.total_exercises += 1

    db.commit()

# Create/Update DailyStats
def update_daily_stats(db, user_id, data, score):
    today = date.today()

    stats = db.query(DailyStats).filter_by(
        user_id=user_id,
        date=today
    ).first()

    if not stats:
        stats = DailyStats(
            user_id=user_id,
            date=today,
            total_workouts=0,
            total_duration=0,
            total_score=0,
            total_steps=0,
            avg_intensity=0,
            avg_stability=0,
            streak_day=1
        )
        db.add(stats)

    # Aggregate
    stats.total_workouts += 1
    stats.total_duration += data.duration
    stats.total_score += score
    stats.total_steps += data.total_steps or 0

    # Running averages
    count = stats.total_workouts

    stats.avg_intensity = (
        (stats.avg_intensity * (count - 1) + (data.avg_intensity or 0)) / count
    )

    stats.avg_stability = (
        (stats.avg_stability * (count - 1) + (data.avg_stability or 0)) / count
    )

    db.commit()
```

### Check this

```python
from datetime import datetime, timedelta

def get_or_create_session(db, user_id, program_id=None):
    session = db.query(WorkoutSession).filter_by(
        user_id=user_id,
        status="active"
    ).order_by(WorkoutSession.created_at.desc()).first()

    if session:
        return session  # ✅ reuse

    # ❗ create new session
    new_session = WorkoutSession(
        user_id=user_id,
        program_id=program_id,
        status="active"
    )

    db.add(new_session)
    db.commit()
    db.refresh(new_session)

    return new_session
```

### React

```bash
expo install expo-sensors

# Full Sensor Service + Payload (ONE FILE)
// sensorService.js

import { Accelerometer, Gyroscope, Pedometer } from 'expo-sensors';

class SensorService {
  accelData = [];
  gyroData = [];
  steps = 0;

  accelSub = null;
  gyroSub = null;
  stepSub = null;

  // 🧠 For live balance
  balanceCallback = null;
  prevMovement = 0;

  // ▶️ START collecting
  async start() {
    this.accelData = [];
    this.gyroData = [];
    this.steps = 0;
    this.prevMovement = 0;

    Accelerometer.setUpdateInterval(500);
    Gyroscope.setUpdateInterval(500);

    // 📊 Accelerometer
    this.accelSub = Accelerometer.addListener(data => {
      this.accelData.push({
        x: data.x,
        y: data.y,
        z: data.z,
        timestamp: Date.now()
      });
    });

    // 🌀 Gyroscope (used for balance)
    this.gyroSub = Gyroscope.addListener(data => {
      const point = {
        x: data.x,
        y: data.y,
        z: data.z,
        timestamp: Date.now()
      };

      this.gyroData.push(point);

      // 🔥 LIVE BALANCE CALCULATION
      const movementRaw =
        Math.abs(data.x) + Math.abs(data.y) + Math.abs(data.z);

      const movement = this.smoothMovement(movementRaw);

      const status = this.getBalanceStatus(movement);

      // 🎯 send to UI
      if (this.balanceCallback) {
        this.balanceCallback(status);
      }
    });

    // 🚶 Steps
    this.stepSub = Pedometer.watchStepCount(result => {
      this.steps = result.steps;
    });
  }

  // ⏹ STOP collecting
  stop() {
    this.accelSub?.remove();
    this.gyroSub?.remove();
    this.stepSub?.remove();

    return {
      accelData: this.accelData,
      gyroData: this.gyroData,
      steps: this.steps
    };
  }

  // 🎣 Subscribe to live balance updates
  onBalanceUpdate(callback) {
    this.balanceCallback = callback;
  }

  // 🧮 Smooth sensor noise
  smoothMovement(current) {
    const smoothed = this.prevMovement * 0.7 + current * 0.3;
    this.prevMovement = smoothed;
    return smoothed;
  }

  // 🎯 Convert movement → human status
  getBalanceStatus(movement) {
    if (movement < 0.5) {
      return {
        label: "🟢 Very good",
        level: 3,
        color: "green"
      };
    } else if (movement < 1.5) {
      return {
        label: "🟡 Normal",
        level: 2,
        color: "orange"
      };
    } else {
      return {
        label: "🔴 Need more effort",
        level: 1,
        color: "red"
      };
    }
  }

  // 🧮 CALCULATE METRICS
  calculateMetrics(accelData, gyroData) {
    let totalIntensity = 0;
    let totalStability = 0;

    accelData.forEach(d => {
      totalIntensity += Math.abs(d.x) + Math.abs(d.y) + Math.abs(d.z);
    });

    gyroData.forEach(d => {
      totalStability +=
        1 / (Math.abs(d.x) + Math.abs(d.y) + Math.abs(d.z) + 0.1);
    });

    return {
      avgIntensity: accelData.length
        ? Number((totalIntensity / accelData.length).toFixed(2))
        : 0,

      avgStability: gyroData.length
        ? Number((totalStability / gyroData.length).toFixed(2))
        : 0
    };
  }

  // 📦 PREPARE FINAL PAYLOAD
  preparePayload({
    programExerciseId,
    sessionId,
    duration,
    reps,
    intensity
  }) {
    const MAX_POINTS = 200;

    const accelData = this.accelData.slice(-MAX_POINTS);
    const gyroData = this.gyroData.slice(-MAX_POINTS);

    const { avgIntensity, avgStability } =
      this.calculateMetrics(accelData, gyroData);

    return {
      program_exercise_id: programExerciseId,
      session_id: sessionId,

      duration,
      reps: reps || null,
      intensity,

      avg_intensity: avgIntensity,
      avg_stability: avgStability,
      total_steps: this.steps,

      sensor_data: {
        accelerometer: accelData.map(d => ({
          x: Number(d.x.toFixed(4)),
          y: Number(d.y.toFixed(4)),
          z: Number(d.z.toFixed(4)),
          timestamp: d.timestamp
        })),
        gyroscope: gyroData.map(d => ({
          x: Number(d.x.toFixed(4)),
          y: Number(d.y.toFixed(4)),
          z: Number(d.z.toFixed(4)),
          timestamp: d.timestamp
        }))
      }
    };
  }
}

// ✅ Export singleton
export default new SensorService();


# Usage Example (VERY IMPORTANT)
import SensorService from './sensorService';
import { useState, useEffect } from 'react';

const [balance, setBalance] = useState("Checking...");

useEffect(() => {
  SensorService.onBalanceUpdate((status) => {
    setBalance(status.label);
  });
}, []);

# ▶️ Start
await SensorService.start();

⏹ Finish
SensorService.stop();

const payload = SensorService.preparePayload({
  programExerciseId: 12,
  sessionId: 5,
  duration: 60,
  reps: 10,
  intensity: 4
});
```
