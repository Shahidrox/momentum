# 🏆 Moment App - Phase 2 (Gamification & Engagement)

Phase 2 transforms the app from a **tracker** into an **engaging habit-building platform**.

This phase introduces:

- Leaderboards
- Challenges
- Badges / Achievements
- Product-based scoring
- Advanced analytics

**🎯 Phase 2 Goals**

- Increase **daily active users (DAU)**
- Improve **retention (streaks & rewards)**
- Encourage **product usage**
- Add **social + competition elements**

**🧱 New Database Tables**

We introduce:

- Challenge
- UserChallenge
- Leaderboard
- Badge
- UserBadge

**🏁 Challenge System**

**Challenge Table**

class Challenge(Base):  
\_\_tablename\_\_= "challenges"  
<br/>id = Column(Integer, primary_key=True)  
<br/>name = Column(String, nullable=False)  
description = Column(String)  
<br/>challenge_type = Column(String)  
\# e.g. "streak", "duration", "steps", "program"  
<br/>target_value = Column(Integer)  
\# e.g. 7 days, 5000 steps, 60 minutes  
<br/>duration_days = Column(Integer)  
<br/>reward_points = Column(Integer, default=0)  
<br/>start_date = Column(Date)  
end_date = Column(Date)

**UserChallenge Table**

class UserChallenge(Base):  
\_\_tablename\_\_= "user_challenges"  
<br/>id = Column(Integer, primary_key=True)  
<br/>user_id = Column(Integer, ForeignKey("users.id"))  
challenge_id = Column(Integer, ForeignKey("challenges.id"))  
<br/>progress = Column(Float, default=0)  
is_completed = Column(Boolean, default=False)  
<br/>started_at = Column(DateTime, default=datetime.utcnow)  
completed_at = Column(DateTime, nullable=True)

**🏆 Leaderboard System**

**Leaderboard Table**

class Leaderboard(Base):  
\_\_tablename\_\_= "leaderboards"  
<br/>id = Column(Integer, primary_key=True)  
<br/>user_id = Column(Integer, ForeignKey("users.id"))  
<br/>score = Column(Float, default=0)  
<br/>period = Column(String)  
\# "daily", "weekly", "monthly"  
<br/>created_at = Column(DateTime, default=datetime.utcnow)

**Leaderboard Logic**

- Daily → based on DailyStats.total_score
- Weekly → sum of last 7 days
- Monthly → sum of last 30 days

**🎖️ Badge System**

**Badge Table**

class Badge(Base):  
\_\_tablename\_\_= "badges"  
<br/>id = Column(Integer, primary_key=True)  
<br/>name = Column(String)  
description = Column(String)  
<br/>condition_type = Column(String)  
\# e.g. "streak", "total_score", "program_complete"  
<br/>condition_value = Column(Integer)

**UserBadge Table**

class UserBadge(Base):  
\_\_tablename\_\_= "user_badges"  
<br/>id = Column(Integer, primary_key=True)  
<br/>user_id = Column(Integer, ForeignKey("users.id"))  
badge_id = Column(Integer, ForeignKey("badges.id"))  
<br/>earned_at = Column(DateTime, default=datetime.utcnow)

**⚙️ Challenge Progress Logic**

Update challenge progress when workout is logged:

def update_challenge_progress(db, user_id, workout):  
challenges = db.query(UserChallenge).filter_by(  
user_id=user_id,  
is_completed=False  
).all()  
<br/>for uc in challenges:  
challenge = uc.challenge  
<br/>if challenge.challenge_type == "duration":  
uc.progress += workout.duration  
<br/>elif challenge.challenge_type == "streak":  
uc.progress += 1  
<br/>if uc.progress >= challenge.target_value:  
uc.is_completed = True  
uc.completed_at = datetime.utcnow()  
<br/>db.commit()

**🏆 Leaderboard Update Logic**

def update_leaderboard(db, user_id):  
from datetime import timedelta, date  
<br/>today = date.today()  
<br/>week_start = today - timedelta(days=7)  
<br/>total_score = db.query(func.sum(DailyStats.total_score)).filter(  
DailyStats.user_id == user_id,  
DailyStats.date >= week_start  
).scalar() or 0  
<br/>entry = Leaderboard(  
user_id=user_id,  
score=total_score,  
period="weekly"  
)  
<br/>db.add(entry)  
db.commit()

**🎖️ Badge Assignment Logic**

def assign_badges(db, user_id):  
stats = get_user_stats(db, user_id)  
<br/>badges = db.query(Badge).all()  
<br/>for badge in badges:  
if badge.condition_type == "streak":  
if stats.streak >= badge.condition_value:  
grant_badge(db, user_id, badge.id)  
<br/>elif badge.condition_type == "total_score":  
if stats.total_score >= badge.condition_value:  
grant_badge(db, user_id, badge.id)

**🛍️ Product-Based Scoring**

Enhance engagement with Naboso products.

**Idea:**

- Assign bonus score when product-linked exercise is performed

def apply_product_bonus(score, product_used):  
if product_used:  
return score \* 1.1 # +10%  
return score

**📈 Advanced Metrics**

Add insights:

- Weekly consistency
- Average intensity
- Program completion %
- Product usage frequency

**📊 APIs to Implement**

**Challenges**

- GET /challenges
- POST /challenges/join
- GET /user/challenges

**Leaderboard**

- GET /leaderboard?period=weekly

**Badges**

- GET /badges
- GET /user/badges

**🎯 Example Challenges (Japan-Friendly 🇯🇵)**

- 7日間チャレンジ (7-day challenge)
- 毎日10分バランス (10 min balance daily)
- 5000歩ウォーキング (5000 steps walking)
- オフィスケア習慣 (office care habit)

**🔥 Key Benefits**

- ✅ Increases retention (streaks, rewards)
- ✅ Encourages daily engagement
- ✅ Promotes product usage
- ✅ Builds habit loops
- ✅ Enables social competition
