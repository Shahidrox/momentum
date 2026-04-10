✅ Core use cases:
- ⏰ Daily reminder
- 🔥 Streak reminder
- 📊 Progress feedback
- 🏁 Workout completed
- 🧠 Smart coaching (future AI)

### ⚛️ 📱 1. React Native Setup (Expo)

Use:

👉 expo-notifications

📦 Install

```bash
expo install expo-notifications
```

### ⚙️ 2. Basic Setup
```bash
import * as Notifications from 'expo-notifications';

// Ask permission
export const requestNotificationPermission = async () => {
  const { status } = await Notifications.requestPermissionsAsync();
  return status === 'granted';
};

# ⏰ 3. Schedule Daily Reminder
export const scheduleDailyReminder = async () => {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: "Time to move 👣",
      body: "Keep your streak alive with a quick exercise!"
    },
    trigger: {
      hour: 9,
      minute: 0,
      repeats: true
    }
  });
};

# 🔥 4. Smart Streak Reminder
export const sendStreakReminder = async () => {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: "🔥 Don’t break your streak!",
      body: "You're doing great — just 5 minutes today!"
    },
    trigger: null // send immediately
  });
};

# 🏁 5. Workout Completion Notification
export const notifyWorkoutComplete = async (score) => {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: "🎉 Great job!",
      body: `You scored ${score} today — keep improving!`
    },
    trigger: null
  });
};
```

---------------------------------------------------
### expo install expo-notifications

```bash
import * as Notifications from 'expo-notifications';

export const registerForPushNotifications = async () => {
  const { status } = await Notifications.requestPermissionsAsync();

  if (status !== 'granted') {
    alert('Permission not granted!');
    return;
  }

  const tokenData = await Notifications.getExpoPushTokenAsync();
  return tokenData.data;
};

# 📡 Send Token to Backend
const sendTokenToBackend = async (token) => {
  await fetch("https://your-api.com/devices", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      user_id: 1,
      push_token: token
    })
  });
};

# ✅ Usage
const initNotifications = async () => {
  const token = await registerForPushNotifications();
  if (token) {
    await sendTokenToBackend(token);
  }
};


```
